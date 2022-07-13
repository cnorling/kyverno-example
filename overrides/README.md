# ABOUT OVERRIDES

Think of overrides that are changing how the given rule behaves. A good use case for an override is to toggle a rule to enforce rather than audit. [this override](/kyverno-example/overrides/enforce-base-rules.yml) and its [definition in kustomization.yml](/kyverno-example/kustomization.yml) is a good example of what an override looks like.

## the problem of patching into array objects

you can read more about this problem [here](/kyverno-example/exceptions/README.md)
