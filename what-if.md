# What if I need to...?

This document describes possible operations that developers may need to perform.

<br>

# Integrate new municipality

Adding a new municipality works mostly by copy-pasting city-specific manifests and by replacing any reference to the city.

### [waltti-apc-deployment](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-deployment)
- Copy manifests in `manifests/*/<city>`
- Add pulsar topics to `streamnative/environment/<env>/topics_source.tf` and apply the terraform.
- Update `FEED_MAP` in `manifests/journey-matcher/overlays/<env>/kustomization.yaml` and `manifests/vehicle-position-splitter/<city>/overlays/<env>/kustomization.yaml`
- Update `PULSAR_CATALOGUE_READERS` in `manifests/vehicle-anonymization-profiler/overlays/<env>/kustomization.yaml`

### [waltti-apc-gcp-terraform](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-gcp-terraform)
- Copy ArgoCD settings in `k8s/argocd-settings/` and apply them to kubernetes.

### [waltti-apc-anonymizer](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-anonymizer)
- `README.md` contains instructions on bypassing the vehicle registry on early stages of municipality integration, see `IS_INITIAL_PROFILE_READING_REQUIRED` and `PROFILE_COLLECTION_BASE`

### [waltti-apc-journey-matcher](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-journey-matcher)
- If the load for journey-matcher gets too high, consider separating it per-municipality

<br>

# Add new vendor

- Send the vendor the [vehicle specification](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-vehicle-specification) and give them a `<vendor-id>`.
- Create staging credentials for MQTT first. Create production credentials only when the staging data is okay.
- Give the user permissions on the vhost `apc`, the same as the existing `telia` user (`rabbitmqctl list_permissions -p apc`).
- Set the topic permissions so that they can only see their own topic subtree `^apc-from-vehicle\.v[0-9][0-9]*\.fi\.waltti\.<vendor-id>\..*$`. In staging, give read and write permission. In production, give write permission. This is the policy for new vendors: the existing `telia` user also has read permission in production.
- Deliver new MQTT credentials through a secure channel.
- Make sure that the authority or the operator registers the counting systems of the vendor in the vehicle registry, see below.
- Monitor the behaviour, and promote to production if the data is okay.

<br>

# New vehicle gets APC devices

- Waltti-APC does not control the vehicle registry. The authority or the operator maintains it, and `vehicle-registry-poller` reads it from its `HTTP_URL`.
- Ask the authority or the operator to add the `countingSystemId` as `PASSENGER_COUNTER` equipment of the vehicle before the vendor sends data.
  Otherwise journey-matcher drops the messages with "countingSystemId was not mapped to a vehicle".
- Ask for the full `countingSystemId`. Case does not matter. journey-matcher also accepts a registry ID that is only the part after the first `-`, and some existing Telia entries use that form, for example `0009d80670e0` for `JL479-0009d80670e0`.
- A static `COUNTING_SYSTEM_MAP` in journey-matcher does not help for an authority that has a vehicle registry poller: each vehicle catalogue update removes the entries of that authority.

<br>

# One vehicle, multiple vendors

Vehicle should send passenger data only from one counting system.
This should only be relevant in pilot/testing cases where there might be systems from multiple vendors aboard.
For more inforation, see `ACCEPTED_DEVICE_MAP` in [waltti-apc-anonymizer](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-anonymizer)

<br>

# Update Kubernetes

### [waltti-apc-gcp-terraform](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-gcp-terraform)
- You can set the version GKE uses from `environments/<env>/cluster.tf`
- It's recommended to keep development one k8s version ahead, as there is no rollback.
- GCP will force updates eventually, so it's better to handle them in advance.

<br>

# Update ArgoCD

### [waltti-apc-gcp-terraform](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-gcp-terraform)
- Download the preferred install.yaml version from [argocd releases](https://github.com/argoproj/argo-cd/releases/) (pick non-HA) and add it to `k8s/argocd/argocd-install-<version>.yaml`
- Diff the old and new version
- If it looks okay, remove the old file and apply the new installation to kubernetes. Make sure you are on the same cluster as the old argocd installation.

<br>

# Update RabbitMQ

### [waltti-apc-deployment](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-deployment)
- You can set the RabbitMQ version from `cloudamqp/environments/<env>/variables.tf`
- The current provider is very finicky, as it can't update the major/minor versions. To properly update:
    1. Update the RabbitMQ version in CloudAMQP
    1. Change the version in `variables.tf`
    1. Plan and apply
