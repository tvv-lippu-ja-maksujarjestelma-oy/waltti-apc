# CI/CD flow

## Happy path:

### Generic flow
1. Always pull latest edge tag into dev environment
2. Run e2e tests in dev. 
3. e2e tests are separate entity (testing multiple microservices not a single microservice)
4. Pull the latest service versions that passed e2e tests in dev environment into stage environment, for example every Wednesday morning at 09:00 local time.
5. In stage manifests, use sha-* tags instead of edge, though. 

In stage and prod, monitor:

1. log message counts of Pulsar clients for different error levels
2. message rates of different Pulsar topics
3. Pulsar topic storage usage
4. cloud resource consumption

If stage or prod has problems according to monitoring, notify developers.

If stage has not had problems for e.g. a week, manually copy stage image versions into prod and update manifest repo tagprod.

### Unhappy path

Our services should survive if external services go down.

If we write broken logic, before reaching prod the logic has to survive:

1. CI tests, e.g. type checking, linting and unit tests,
2. e2e tests in dev, e.g. a week in stage without tripping alarms.

If the broken logic breaks the data flow so that a downstream microservice chokes on broken messages it consumes from Pulsar, in dev and stage it is probably easiest to fix the logic bug and forcefully empty the relevant Pulsar topics with broken messages.

If the broken logic reaches prod, we might need to fix the downstream service with a kludge.