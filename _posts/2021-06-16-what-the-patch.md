---
layout: post
title: "What the patch?!"
description: "The differences between Kustomize's patches, patchesJson6902 and patchesStrategicMerge"
---

At work I deal with Kubernetes and kustomize on a daily basis.
Today I have pulled my hair about how I could patch a value in a job that was included in the base kustomization.
I knew that there were several ways to do it, and I realized that I could not tell the difference between `patches`, `patchesJson6902` and `patchesStrategicMerge`.

It turns out that the three directives can be regarded as variations of the same patching mechanism.
They differ in terms of customizability, and hence the number of things you can do (and break) with them.
To me, the hierarchy feels as follows, starting with the least versatile and powerful directive:
3. `patchesStrategicMerge`
2. `patchesJson6902`
1. `patches`

---

Why is `patchesStrategicMerge` the least powerful and versatile?
Because it's simple! Try to use it, the reviewers will be thankful.

Check the [documentation here](https://github.com/kubernetes-sigs/cli-experimental/blob/e8661e62fbff9bb41703e663c5d6f9730f121a16/site/content/en/references/kustomize/kustomization/patchesStrategicMerge/_index.md).
It tells us that *each patch should be either a relative file path or an inline content resolving to a partial or complete resource definition*. 
That means these patches do what we are used to from git, for example. 
They patch a single thing, so are ideal for correcting environment variables, names, paths, and the likes.

---

It turns out that the `patchJson6902` directive is an older keyword, that must also target a specific resource,
but can deliver more complex operations on the patch target.
These operations are specified in the JSON6902 format, hence the name.
The actual patch is delivered via JSON (but can also be YAML), for instance like below.
More examples can be found in the [documentation](Documentation: https://github.com/kubernetes-sigs/cli-experimental/blob/e8661e62fbff9bb41703e663c5d6f9730f121a16/site/content/en/references/kustomize/kustomization/patchesjson6902/_index.md).
```json
[
  {
    "op": "remove",
    "path": "/spec/template/spec/containers/0/env/0/valueFrom"
  },
  {
    "op": "add",
    "path": "/spec/template/spec/containers/0/env/0/value",
    "value": "SOMEVALUE"
  }
]
```

Finally, the `patches` directive is the new shiny swiss army knife.
It needs a target, too, but the target can be a *regex*.
The patch instructions are delivered in the JSON6902 format introduced above.
In short, this directive allows you to patch multiple targets using complex operations all in a single directive.
Truly powerful, and truly tricky to review.
You can find the [whole documention for this directive here](https://github.com/kubernetes-sigs/cli-experimental/blob/e8661e62fbff9bb41703e663c5d6f9730f121a16/site/content/en/references/kustomize/kustomization/patchesStrategicMerge/_index.md).

### Sources
* https://github.com/kubernetes-sigs/kustomize/issues/2705
* https://stackoverflow.com/questions/63604579/what-is-the-difference-between-patches-vs-patchesjson6902-in-kustomize