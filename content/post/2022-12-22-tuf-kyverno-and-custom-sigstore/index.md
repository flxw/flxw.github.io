+++
date = 2022-12-22
title = "TUF, Kyverno and a private Sigstore"
description = "Improving the Kyverno integration with a custom Sigstore deployment"
tags = ["sigstore"]
toc = true
+++

Merry Christmas everyone! Here's another follow-up post - this time on the previous post on the [integration of Kyverno and a custom Sigstore deployment](/integrating-kyverno-with-a-private-sigstore-deployment.html).
Sigstore is a relatively new technology, so many things in the ecosystem are still in motion.
This post dives into internals of the Kyverno helm chart and `cosign`, a Sigstore client.

Kyverno is an admission controller for Kubernetes.
It can verify whether images that are about to be deployed match certain requirements.
These requirements are specified in so-called *policies*.
One such policy could require images to be signed by a certain person or to have provenance information attached.
This policy would bank on Sigstore, and `kyverno` would use `cosign` internally to process it.

Per default, `cosign` communicates with the public Sigstore deployment, and requires additional initialization to communicate with a different deployment.
In the previous post, this initialization was circumvented by passing the required public keys in environment variables.

## Providing public keys is fine for testing
As shown in the *Future Work* section of the preceding article, public keys are a great way to get going with a test on a larger scale.
Maintainability remains a major concern, however.
A key rotation on any of the Sigstore components requires an update of the distributed keys, and that will cause Kyverno to be unable to verify signatures on artifacts that were signed using old keys. 

This problem is addressed by [The Update Framework (TUF)](https://theupdateframework.io/).
Sigstore uses TUF as a distribution mechanism for its keys.
Understanding TUF is hard, and this [post by Dan Lorenc](https://blog.sigstore.dev/the-update-framework-and-you-2f5cbaa964d5) can help you get started.

## How cosign uses TUF
When verifying signatures, `cosign` relies on information contained inside the `~/.sigstore/root` directory.
If this directory does not exist, `cosign` initializes a local copy of the public TUF repository.
As of December 22 2022, the contents look as follows:

```bash
λ  ~  tree ~/.sigstore/root
root
├── remote.json
├── targets
│   ├── artifact.pub
│   ├── ctfe.pub
│   ├── ctfe_2022.pub
│   ├── fulcio.crt.pem
│   ├── fulcio_intermediate_v1.crt.pem
│   ├── fulcio_v1.crt.pem
│   └── rekor.pub
└── tuf.db
    ├── 000002.log
    ├── CURRENT
    ├── CURRENT.bak
    ├── LOCK
    ├── LOG
    └── MANIFEST-000003

2 directories, 14 files
```

When using a custom Sigstore deployment, cosign needs to be explicitly initialized with the custom TUF information.
On the command line this is done with the following command:
```
cosign initialize --root root.json --mirror https://<some-tuf-mirror>
```

If the `~/.sigstore/root` directory already exists, `cosign` will try to use it and also update its contents to match those of the TUF mirror.

## How cosign works inside of Kyverno
`Kyverno` runs inside of Kubernetes pods, and it uses the `cosign` Go libraries. Those are the same code routines as the command-line client.
That means that `cosign` has to have a directory inside the `kyverno` container to create the local TUF repository and keep it up to date.

This is the entry point to a solution.
The `kyverno` helm chart defines an environment variable named `TUF_ROOT`, and points it to an empty directory. See the source code [here](https://github.com/kyverno/kyverno/blob/main/charts/kyverno/templates/deployment.yaml#L152).
The variable is picked up inside of the `sigstore` Go library used by `cosign` internally - see the code [here](https://github.com/sigstore/sigstore/blob/abdf5cf3faa5f93af333376aaccba4e96ee7a242/pkg/tuf/client.go#L49).

The provided directory is empty but writeable, so that `cosign` can place the TUF repository inside it, and keep it up to date. *Empty* is the key word.

In the next section I will demonstrate an approach to provide a `TUF_ROOT` directory that isn't empty and has valid contents instead.

# Providing a TUF root in Kyverno
The goal is to provide a prepopulated directory to the `kyverno` container.
Using a persistent volume sounds like a simple solution, but might invoke concurrency issues.
Given that a `kyverno` deployment usually consists of several pods, two pods might try to update the local TUF repository at the same time.

I believe that it is better to accept the cost of repeated updates, and use a separate volume for each container.
As mentioned above, the `kyverno` helm chart provides an `emptyDir` volume to the `kyverno` pods.
We shall patch the `kyverno` deployment, and use an `initContainer` to initialize a local TUF repository on the mounted volume.
Once the actual container with `kyverno` inside comes up, `cosign` will find all it needs inside the provided directory.

This is the patch for the `kyverno` deployment:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kyverno
  namespace: kyverno
spec:
  template:
    spec:
      initContainers:
      - args:
        - cosign initialize --root $TUF_URL/root.json --mirror $TUF_URL
        command:
        - sh
        - -c
        env:
        - name: TUF_URL
          value: https://path.to.your.tuf.repository.com
        image: ghcr.io/sigstore/cosign/cosign:v1.13.1
        name: init-tuf-root
        volumeMounts:
        - mountPath: /home/nonroot/.sigstore/root
          name: sigstore
```

## Verdict
The presented approach allows one to look more confidently at the integration of Sigstore and Kyverno.
If a pre-existing Kyverno deployment should be used, this approach is fairly simple to implement using `kustomize`.

However, the [policy controller provided by the Sigstore project](https://docs.sigstore.dev/policy-controller/installation) offers an integration that requires less plumbing.
Its configuration simply accepts pointers to the TUF root and the TUF mirror, letting it handle the rest of the initialization.
