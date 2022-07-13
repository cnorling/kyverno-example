# ABOUT EXCEPTIONS

when first implementing a policy, it's expected that there should be some violations found in your environment. It's equally expected that you will have to whitelist certain entities

## Adding exceptions

To implement an exception, you can create a JSON patch and apply it to your targeted resource. Exceptions are categorized in their own individual folders because a single rule might have more than one exception. Don't forget to inspect how the exception is defined in the base layer [kustomization.yml](/kyverno-policies/base/kustomization.yml). You should always add a comment that backlinks to any resources that explain why the exception is needed.

## The problem of patching into array objects

Kustomize is capable of adding additional maps to an array with a simple overlay. However, using a [strategic merge](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-api-machinery/strategic-merge-patch.md) on an existing item in an array will only use the highest defined layer. Any keys present in the base layer will be deleted. Because Kyverno uses an array of rules, this becomes a problem you have to tackle. You could always just add the whole defined rule in the uppermost layer, but this isn't a practical solution. There are two potential ways to patch into an array:

### Method 1

It's possible to just include the whole rule in the uppermost layer, but this defeats the purpose of digesting rules from a remote location. You now have to update your layer with new changes, and it removes the possibility of having more than one exception or override modify the rule in question.

### Method 2

The risk with this method is that [json patches](https://datatracker.ietf.org/doc/html/rfc6902#appendix-A.2) might not always be available in kustomize. The team responsible for maintaining kustomize also [has no plans to resolve this problem space outside of json patches](https://github.com/kubernetes-sigs/kustomize/issues/3265). I was unable to find a more elegant solution than json patches; so be it.

You _can_ express a JSON patch as YAML (it will still be parsed as JSON as YAML is a superset of JSON). It's a simple operation you can read about [here](https://github.com/kubernetes-sigs/kustomize/blob/master/examples/jsonpatch.md). To form your JSON patch, you will have to navigate down the tree, then specify the array index you want to contribute to. It looks like this in an exception:

```
- op: add
  path: /spec/rules/0/exclude
  value:
    all:
      - resources:
        names:
          - "ingress-nginx-controller*"
```

You can also create a new array object to add to the ruleset by using the hyphen character where an array index would be present. Here's an example:

```
- op: add
  path: /spec/rules/-
  value:
    exclude:
      resources:
        name: "istio-proxy*"
        kinds:
          - Pod
        namespaces:
          - istio-system
          - istio-gateway
```
