# What if I need to...?

This document describes possible operations that developers may need to perform.

## Integrating a new city

Adding a new city works mostly by copy-pasting city-specific manifests and by replacing any reference to the city.

### [waltti-apc-deployment](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-deployment)
- Copy manifests in `manifests/*/<city>`
- Update `FEED_MAP` in `manifests/journey-matcher/overlays/<env>/kustomization.yaml`
- Add pulsar topics to `streamnative/environment/<env>/topics_source.tf` and apply the terraform.

### [waltti-apc-gcp-terraform](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-gcp-terraform)
- Copy ArgoCD settings in `k8s/argocd-settings/` and apply them to kubernetes.
