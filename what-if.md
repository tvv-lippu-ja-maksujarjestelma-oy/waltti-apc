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

- Create staging and production credentials for MQTT.
- Set the topic permissions so that they can only see their own topic subtree
- Vendors should initially only have write permission to staging.
- Deliver new MQTT credentials through a secure channel.
- Monitor the behaviour, and promote to production if the data is okay.

<br>

# New vehicle gets APC devices

- Nothing needs to be done

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

# Update RabbitMQ

### [waltti-apc-deployment](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-deployment)
- You can set the RabbitMQ version from `cloudamqp/environments/<env>/variables.tf`
- The current provider is very finicky, as it can't update the major/minor versions. To properly update:
    1. Update the RabbitMQ version in CloudAMQP
    1. Change the version in `variables.tf`
    1. Plan and apply
