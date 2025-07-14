+++
date = 2022-12-06
title = "Kyverno and a custom Sigstore"
description = "Instructions for using Kyverno with a custom Sigstore deployment"
tags = ["sigstore"]
+++

This is a follow-up post to my previous writing on [standing up a local Sigstore deployment](/standing-up-sigstore-locally.html).
This post is quite technical and requires a firm understanding of how Sigstore and its components work.
See the [Sigstore project page](https://sigstore.dev) for pointers for diving into the subject.

After deploying Sigstore and pushing signatures and provenance into it for a while, I tried
to make use of this data on a larger scale inside a Kubernetes cluster.
The goal was to verify signatures and provenance information of an image before actually running it.

The Sigstore project offers [policy-controller](https://docs.sigstore.dev/policy-controller/overview) for that, but 
[Kyverno](https://kyverno.io) also implements [signature verification](https://kyverno.io/docs/writing-policies/verify-images/).
`Kyverno` is a very versatile policy manager, while `policy-controller` is specific to Sigstore.
Nonetheless, both allow verifying container image signatures and related metadata before deploying an image,
acting as a [Kubernetes admission controller](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/) in the process.

In the following, I will demonstrate how to work around current `kyverno` limitations and have it verify keyless signatures using self-hosted Sigstore components.

# Recap: Keyless signatures
Keyless signatures are arguably the most innovative feature of Sigstore.
It does away with the dangerous and error-prone management of local key pairs.
Instead, it leverages an identity provider to verify the signers' identity
and embeds this identity into the artifact signing certificate.
After the signing act, the signing certificate gets thrown away, and the only remnant is the public key inside the Rekor entry.

# A policy for public Sigstore
This section showcases an example `kyverno` policy.
It validates signatures on any image that matches one of the regexes in `imageReferences`,
and requires them to be signed keyless by `some-email@protonmail.com`, courtesy of `some-idp`.
It will only log policy failures, as specified by the value of `validationFailureAction`:

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: check-container-signature
spec:
  validationFailureAction: audit
  rules:
    - name: check-container-signature
      match:
        any:
        - resources:
            kinds:
              - Pod
      verifyImages:
      - imageReferences:
        - "ghcr.io/your/image*"
        attestors:
        - entries:
          - keyless:
              subject: "some-email@protonmail.com"
              issuer: "some-idp"
```

The execution of this policy will invoke `cosign`, as demonstrated by this [code snippet](https://github.com/kyverno/kyverno/blob/3a8affab1600f28e097b1cfa5c8c94b62df7e573/pkg/engine/imageVerify.go#L440).
The `cosign` tool will then contact the public Sigstore deployment, and check the signatures contained inside the public Rekor instance.
This works beautifully.

It works, because cosign is configured by default to speak to the public Sigstore deployment.
The verification requires having access to the correct set of public keys for Rekor, Fulcio and ctlog.
As we want to use the policy with a private Sigstore deployment, `cosign` inside of Kyverno needs to be informed about these.

Unfortunately, it is not yet possible to do this within Kyverno as the next section will show.

# Problems with the current Kyverno implementation
As of Kyverno 1.8.2, the advanced options for verifying keyless signatures with a custom deployment are not documented and are hidden in the [source code](https://github.com/kyverno/kyverno/blob/3a8affab1600f28e097b1cfa5c8c94b62df7e573/pkg/engine/imageVerify.go#L552-L562).
The code reveals that it is possible to point the policy to a specific Rekor instance and provide the public Fulcio key in PEM format.

```yaml
# to be added as part of a `keyless` entry
rekor:
    url: https://url.to.your.rekor.com/make/sure/its/accessible
roots: |
    PEM OF THE FULCIO ROOT
```

Unfortunately, this is only a third of the required number of public keys and certificates.
Trying out a policy with the options from above leads to a complaint about cosign not being able
to verify the signed certificate timestamps (SCT):

```bash
kubectl run test --image=ghcr.io/flxw/example-image@sha256:cab1dab84c35c2f9382ece97b02a42903a3135e7c1b81d937798bde9bf3ef486

Error from server: admission webhook "mutate.kyverno.svc-fail" denied the request: 

policy Pod/default/test for resource violation: 
check-container-signature:
  check-container-signature: |-
    failed to verify image ghcr.io/flxw/example-image@sha256:cab1dab84c35c2f9382ece97b02a42903a3135e7c1b81d937798bde9bf3ef486: .attestors[0].entries[0].keyless: no matching signatures:
    ctfe public key not found for embedded SCT
```

The SCTs are doled out by ctlog, and nowhere have we passed in its public key.
The same goes for the Rekor public key, which is required to verify the signature on the Rekor entry.
Failure should not be a surprise then.

In the following, I will present a fix for this issue.

# A policy for a custom Sigstore
As demonstrated in the previous section, the `cosign` tool used by Kyverno requires all Fulcio, Rekor, and ctlog signing information to successfully work with non-standard Sigstore components.
Unfortunately, they can not be passed to the policy directly as of now.
This problem is also partially tracked in this [GitHub issue](https://github.com/kyverno/kyverno/issues/5165).

Thankfully, `cosign` can be made to work with [custom components](https://docs.sigstore.dev/cosign/custom_components) via environment variables, aptly named `SIGSTORE_REKOR_PUBLIC_KEY` and `SIGSTORE_CT_LOG_PUBLIC_KEY_FILE`.
These variables contain file locations that the tool will retrieve the public keys from.
The files shall be mounted in the respective pods as files, created from a configmap:

```bash
#!/bin/bash
curl https://your-rekor.com/api/v1/log/publicKey > rekor-public-key.pem

kubectl get secret/ctlog-public-key \
    -n ctlog-system \
    -o jsonpath='{.data.public}' | base64 -d > ctlog-public-key.pem

kubectl create cm public-keys \
    -n kyverno \
    --from-file=rekor=rekor-public-key.pem \
    --from-file=ctlog=ctlog-public-key.pem
```

The environment variables need to be set on the pods where `cosign` is executed - every `kyverno` pod.
Unfortunately, this forces us to add the two environment variables to the `kyverno` deployment manually:

```yaml
# add these to the kyverno deployment
[...]
      volumes:
      - configMap:
          name: public-keys
        name: public-keys
[...]
        volumeMounts:
        - mountPath: /public-keys
          name: public-keys
[...]
        env:
        - name: SIGSTORE_CT_LOG_PUBLIC_KEY_FILE
          value: /public-keys/ctlog
        - name: SIGSTORE_REKOR_PUBLIC_KEY
          value: /public-keys/rekor
[...]
```

Now, you are ready to debug and test whether the changes have the desired effect.
I recommend starting a container image and provoking a policy failure.
This will generate output in response to the `kubectl run` command.
An example would be a signature by `wolff.felix@protonmail.com` when the policy requires one by `wolff.peter@protonmail.com`.
Here's an example of the error message you would receive with an enforcing policy:

```
kubectl run test --image=ghcr.io/flxw/example-image@sha256:cab1dab84c35c2f9382ece97b02a42903a3135e7c1b81d937798bde9bf3ef486
Error from server: admission webhook "mutate.kyverno.svc-fail" denied the request: 

policy Pod/default/test for resource violation: 

check-container-signature:
  check-container-signature: 'failed to verify image ghcr.io/flxw/example-image@sha256:cab1dab84c35c2f9382ece97b02a42903a3135e7c1b81d937798bde9bf3ef486:
    .attestors[0].entries[0].keyless: subject mismatch: expected wolff.peter@protonmail.com,
    received wolff.felix@protonmail.com'
Error from server: admission webhook "mutate.kyverno.svc-fail" denied the request: 
```

If you have received a similar error message, it means that `cosign` and `kyverno` can work together and successfully verify signatures and match them against the policy.
Now it is time for the reverse test. Start a compliant container image in a pod.
There won't be any output for you to see.

If you check the logs on the `kyverno` deployment, you will find a message similar to the one below:

```
kyverno I1206 12:25:02.344950       1 cosign.go:92] cosign "msg"="verified image" "bundleVerified"=true "count"=1
```

# Future Work
While it is nice to have an operational private deployment along with verification, it is important to realize that the approach presented in this post is no permanent solution. It comes with two huge problems:

First, if `kyverno` was installed using a helm chart, a simple `helm upgrade` can undo the customizations.

Second, a key rotation on any of the Sigstore components will cause Kyverno to be unable to verify signatures on artifacts that were signed using old keys. This is where TUF can help, and provide backwards compatibility.

I shall try in the future to help with solving [kyverno#5165](https://github.com/kyverno/kyverno/issues/5165) and remedy this.

Thanks for reading!
