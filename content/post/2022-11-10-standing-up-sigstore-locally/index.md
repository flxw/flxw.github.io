+++
date = 2022-11-10
title = "Standing up Sigstore locally"
description = "Instructions for running a Sigstore deployment on your machine in k8s"
tags = ["sigstore"]
+++

In this post, I describe steps on how to stand up a Sigstore deployment on your Kubernetes cluster,
adding detail and background information where I feel that it is needed.
The post is aimed at the early adopter of the project who is roughly familiar with how Sigstore works internally.
If you follow right through to the end, you will have a working Sigstore deployment on your machine - 
usable in the same manner as you would a remote deployment.

As usual, I stand on the shoulders of giants, as I have benefitted a lot from
[other](https://github.com/sigstore/scaffolding/blob/main/getting-started.md)
[earlier](https://sthw.decodebytes.sh/)
[articles](https://blog.sigstore.dev/scaffolding-sigstore-e893eb962f22) on this subject out there.
I hope that this updated version of a setup guide helps with getting up to speed with Sigstore quickly.

All you need to follow along is a working Docker installation on your machine, and Internet connectivity.
Throughout the next steps, I will sometimes refer to files that aren't printed out here.
You can find them in this repository: [flxw/sigstore-local-setup](https://github.com/flxw/sigstore-local-setup).

# Step 1: Set up KinD and Ingress
Now, we set up our Kubernetes cluster, using kind.
I find it to be an awesome tool for quickly prototyping and testing clusters.
If you are unfamiliar with it, here's the description from their [website](https://kind.sigs.k8s.io/).
Kind is a shorthand for *Kubernetes in docker*:

> kind is a tool for running local Kubernetes clusters using Docker container “nodes”.
> kind was primarily designed for testing Kubernetes itself, but may be used for local development or CI.

Along with the cluster, we shall also set up an nginx ingress, in line with the [official instructions](https://kind.sigs.k8s.io/docs/user/ingress/#ingress-nginx):

```bash
#!/bin/bash
brew install kind cosign ko # only for brew users ;)
kind create cluster --name kind-for-helm --config=kind-cluster-config.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

**Note to Mac users:** * Make sure to have OpenSSL at version 1.1, LibreSSL 2.8.x can cause problems.

# Step 2: Install Sigstore

After checking that our cluster and ingress are ready, we can install the bulk of Sigstore via [helm](https://helm.sh):

```bash
#!/bin/bash
helm repo add sigstore https://sigstore.github.io/helm-charts
helm upgrade \
    -i scaffold \
    sigstore/scaffold \
    -n sigstore \
    --create-namespace \
    --values scaffold.values.yaml
```

This `scaffold` chart bundles individual charts for most of the Sigstore services: fulcio, rekor, ctlog, trillian, and TUF with an underlying MySQL database.
Additionally, it generates all of the required signing keys as secrets, configmaps, services, and ingresses.

# Step 3: Certificate chain and domains
To make Sigstore clients work with the cluster, you need to generate a chain of certificates with a self-signed root.
You then need to add the root certificate to your OS's trust store.
Alternatively, you can work with independent certificates, but I find it easier to add only one certificate to the trust store.
Finally, you need to add four entries to your `/etc/hosts` file.

```bash
#!/bin/sh
# create a self-signed CA certificate (add ca.cert.pem to trust store and configure explicit trust)
openssl req -x509 -newkey rsa:4096 -keyout ca.private.pem -out ca.cert.pem -sha256 -days 365 -nodes

for service_name in rekor fulcio tuf; do
    cat << EOF > $service_name.cert.ext
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = $service_name.sigstore.local
EOF

    openssl req -new -newkey rsa:4096 \
        -keyout $service_name.private.pem \
        -out $service_name.req.pem -nodes

    openssl x509 -req -in $service_name.req.pem \
        -days 365 -CA ca.cert.pem -CAkey ca.private.pem \
        -CAcreateserial -out $service_name.signed.cert.pem \
        -extfile $service_name.cert.ext

    kubectl create secret tls $service_name-tls \
        --namespace=$service_name-system \
        --cert=$service_name.signed.cert.pem \
        --key=$service_name.private.pem
done

cat <<EOF >> /etc/hosts
127.0.0.1 fulcio.sigstore.local
127.0.0.1 rekor.sigstore.local 
127.0.0.1 tuf.sigstore.local
127.0.0.1 registry.local
EOF
```

# Step 4: Cosign initialization
Out of the box, `cosign` is configured to work with the public infrastructure.
Hence, we need to tell it about the certificates used in our cluster.
To do that, tuf compiles a JSON file that needs to be extracted from the cluster.
It only needs to be used once for every initialization.

```bash
#!/bin/bash
kubectl -n tuf-system get secrets tuf-root -ojsonpath='{.data.root}' \
    | base64 -d > root.json
cosign initialize --root root.json --mirror https://tuf.sigstore.local
```

# Step 5: Test

From now on, it's just testing and enjoying what you have built. :)
In our test, we shall put a container into a locally running registry and sign it.
Afterward, we shall verify it.


```bash
#!/bin/bash

# First, we'll spin up the local registry
docker run -d \
    --restart=always \
    -p 5000:5000 \
    --name registry.local \
    registry:2

# Second, a test container is created
# Make sure you have go installed for this
KO_DOCKER_REPO=registry.local:5000/sigstore
pushd $(mktemp -d)
go mod init example.com/demo
cat <<EOF > main.go
package main
import "fmt"
func main() {
   fmt.Println("hello world")
}
EOF
export IMAGE=`ko publish -B example.com/demo`
echo "Created image $IMAGE"
popd

# Third, we shall sign the container
REKOR_URL=https://rekor.sigstore.local
FULCIO_URL=https://fulcio.sigstore.local
export COSIGN_EXPERIMENTAL=1

# add --verbose if you are curious ;)
cosign sign \
  --fulcio-url=$FULCIO_URL \
  --rekor-url=$REKOR_URL \
  --force \
  --allow-insecure-registry \
  $IMAGE

# Fourth and last - signature verification!
cosign verify \
    --allow-insecure-registry \
    --rekor-url=https://rekor.sigstore.local \
    $IMAGE
```

Congratulations - you have now mastered the deployment of Sigstore on your machine!
Running such a setup in production is a bit more complex,
and we are currently hard at work documenting the efforts involved.
If you're curious about what's next and how to help, read on in the next section.

# Getting involved
This is the shortest and fastest way to set up Sigstore I know - until now!
The Sigstore project is just getting started, and there are tons of ways to [contribute](https://docs.sigstore.dev/contributing).
You can meet me and many others in the Sigstore Slack channel `private-sigstore-users`, putting together a manual for operating a deployment for a longer duration.

This is a living document, and I'll edit it in the future to include DEX as an OIDC token forwarder.
