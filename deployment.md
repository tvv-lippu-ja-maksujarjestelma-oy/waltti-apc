# Deployment

## Environments

We have three environments: dev, staging, prod.

dev:

- Our development environment.
- We test and break things by ourselves in dev.
- Runs the latest commit for branch `main` for every microservice.

staging:

- Our quality assurance environment.
- As identical to production as possible.
- The development environments of the other systems we integrate to, e.g. Waltti Raportointi or Waltti Vehicle Registry, connect to staging.
- New APC counting systems or new versions of old APC counting systems connect to staging before they are accepted into production.

prod:

- Our production environment.
- Production data is what is shown to passengers and Waltti customers, e.g. transit planners in municipalities.
- The production environments of the other systems we integrate to, e.g. Waltti Raportointi or Waltti Vehicle Registry, connect to prod.

### Naming

The following table describes the differences in naming between the environments.

|                                   | dev                                          | staging                                     | prod                                        | common                                                             |
| --------------------------------- | -------------------------------------------- | ------------------------------------------- | ------------------------------------------- | ------------------------------------------------------------------ |
| Description                       | Productization phase development environment | Productization phase staging environment    | Productization phase production environment | Common, shared things like DNS management for productization phase |
| Method                            | IaC                                          | IaC                                         | IaC                                         | IaC                                                                |
| GCP project                       | apc-sandbox                                  | apc-staging                                 | apc-prod                                    | apc-sandbox                                                        |
| GCP resource region, e.g. for K8s | europe-west3 (Frankfurt, Germany)            | europe-west3 (Frankfurt, Germany)           | europe-west3 (Frankfurt, Germany)           | europe-west3 (Frankfurt, Germany)                                  |
| K8s cluster                       | prototype                                    | staging                                     | prod                                        | N/A                                                                |
| K8s namespace                     | dev                                          | staging                                     | prod                                        | N/A                                                                |
| RabbitMQ GCP region               | europe-west3 (Frankfurt, Germany)            | europe-west3 (Frankfurt, Germany)           | europe-west3 (Frankfurt, Germany)           | europe-west3 (Frankfurt, Germany)                                  |
| CloudAMQP MQTT hostname           | mqtts-dev.apc.waltti.fi                      | mqtts-staging.apc.waltti.fi                 | mqtts-prod.apc.waltti.fi                     | N/A                                                                |
| MQTT broker CNAME                 | mqtts-dev.apc.waltti.fi                      | mqtts-staging.apc.waltti.fi                 | mqtts-prod.apc.waltti.fi                     | N/A                                                                |

## CI/CD flow

### Happy path:

#### Generic flow

1. Always pull latest edge tag into dev environment
2. Run e2e tests in dev.
3. e2e tests are separate entity (testing multiple microservices not a single microservice)
4. Pull the latest service versions that passed e2e tests in dev environment into staging environment, for example every Wednesday morning at 09:00 local time.
5. In staging manifests, use sha-\* tags instead of edge, though.

In staging and prod, monitor:

1. log message counts of Pulsar clients for different error levels
2. message rates of different Pulsar topics
3. Pulsar topic storage usage
4. cloud resource consumption

If staging or prod has problems according to monitoring, notify developers.

If staging has not had problems for e.g. a week, manually copy staging image versions into prod and update manifest repo tag prod.

### Unhappy path

Our services should survive if external services go down.

If we write broken logic, before reaching prod the logic has to survive:

1. CI tests, e.g. type checking, linting and unit tests,
2. e2e tests in dev, e.g. a week in staging without tripping alarms.

If the broken logic breaks the data flow so that a downstream microservice chokes on broken messages it consumes from Pulsar, in dev and staging it is probably easiest to fix the logic bug and forcefully empty the relevant Pulsar topics with broken messages.

If the broken logic reaches prod, we might need to fix the downstream service with a kludge.
