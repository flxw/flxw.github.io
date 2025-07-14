+++
date = 2022-11-04
title = "Oneliner: External secret decoding"
description = "A reminder how to decode secrets with secrets-manager in k8s"
tags = ["oneliner"]
+++

If you use `external-secrets` to sync secrets from some a key management
system like AWS KMS or Vault into your cluster,
you have become familiar with the `ExternalSecret` object introduced by it.

In case you want to sync a secret that has multiple lines,
you need to encode the secret _inside_ your KMS,
and decode it inside the `ExternalSecret` using a `decodingStrategy`.
The values will be base64-encoded again by k8s when the `Secret` gets created.

An example would be as follows (taken from [here](https://external-secrets.io/v0.6.1/guides/decoding-strategy/)):
```
KMS secret value: aGFwcHkgc3RyZWV0
ExternalSecret decodes it: happy street
k8s Secret data: aGFwcHkgc3RyZWV0
Application reads: happy street
```

Please see the following object definition for a complete definition:
```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: your-access-credentials
spec:
  refreshInterval: 1m
  secretStoreRef:
    name: cluster-secrets-store
    kind: ClusterSecretStore
  target:
    name: your-access-credentials
  dataFrom:
  - extract:
      key: svc/app/secret
      decodingStrategy: Base64
```
