# End to End (E2E testing)

## Introduction

There should be an E2E/integration test after development that does quality testing before promoting the docker images to staging environment. In perfect usage scenarios we would send data and verify that the whole path works. In rest&microservice environments this is easy to achieve via mock, curl, jq tooling. In this perfect scenario we would also need to test pulsar system not just the micro services we are running (simulating latency, misconfigurations, broken systems etc). A good testing solution should contain also some elements of chaos engineering.

There are no proven E2E testing frameworks or methods for streaming environemnts. Pulsar uses testcontainers, Pulsar functions and Java to test itself. Kafka uses similar system. Also there seems to be a kind of trust that SaaS providers will maintain their system so the tests are limited to unit testing a single service. Also there is a concern that if we send fake data for testing purposes into development system that would pollute the actual data. This raises the question of development environemnt? Is it just fake test data? In an ideal case there would be real data going thru the system to see any long term issues such as memory leaks that happen when the system is long running.


# Solutions

There are solutions to this problem. Either use an existing framework or create our own framework for testing E2E pulsar environments. 

## Create a framework

Probably the only reasonable way forward. It would require a sending loop and receiving loop. See Github issue [#302](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc/issues/302) for discussion about this. Implementing this kind of framework/service would be major undertaking.  For comparison HSL has a testing system that has been tailor made. 

There are similar problems with feature branch development ie how we handle big changes. Deploy a new system next to old one, run tests in it, promote? Deploying a new pulsar just testing is an expensive option and hard to do in code. Or just use pulsar containers? Or keep development environment clean and use only test data then clean the system? 


## Use existing framework

Sadly there are no existing testing frameworks for E2E testing pulsar environments. Trace based testing seems promising but it is very young and thus rapidly changing. It would require implemnting an OpenTelemery tracing in the applications. Also it would require installing a telemetry data gathering system. Pulsar supports with caveats tracing but StreamNative Cloud version does not so we would have to add tracing data into our Pulsar json payload.

Pros:
- would increase visibility into whole system
- performance data

Cons:
- yet another Kubernetes operator to be maintained
- Slightly time consuming to setup
- Requires OpenTelemetry knowledge
- Apache Pulsar support for OpenTelemetry is still on the way, especially for node clients
