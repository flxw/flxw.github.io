---
layout: post
title: "Oneliner: Merge kubectl configs"
desc: A oneliner for merging two kubectl configuration files
category: blog
tags: oneliner
---
Today's oneliner is for merging two kubectl configuration files into one.
This can be useful when creating new clusters and exchanging small config files with your colleagues:

```bash
export KUBECONFIG=~/.kube/config.old:~/cluster
mv ~/.kube/config ~/.kube/config.old
kubectl config view --flatten > ~/.kube/config
```

Note the colon that seperates the filepaths in `KUBECONFIG` - you can pass in more than one file.