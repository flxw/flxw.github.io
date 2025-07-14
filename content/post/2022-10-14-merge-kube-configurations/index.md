+++
date = 2022-10-14
title = "Oneliner: Merge kubectl configs"
description = "A oneliner for merging two kubectl configuration files"
tags = ["oneliners"]
+++
Today's oneliner is for merging two kubectl configuration files into one.
This can be useful when creating new clusters and exchanging small config files with your colleagues:

```bash
export KUBECONFIG=~/.kube/config.old:~/cluster
mv ~/.kube/config ~/.kube/config.old
kubectl config view --flatten > ~/.kube/config
```

Note the colon that seperates the filepaths in `KUBECONFIG` - you can pass in more than one file.
