# waltti-apc

The automatic passenger counting (APC) pilot for Waltti.

This main repository documents the high-level structure of the project, links to the other repositories of the project and contains the issues and the Kanban board.

## Architecture

### High-level architecture

```mermaid
flowchart TB

  %% Nodes

  onboard("Onboard counting systems")
  rtpiinput("Waltti RTPI")
  rtpioutput("Waltti RTPI with APC data")
  vehicleregistry("Vehicle registry")
  subgraph azure["Microsoft Azure"]
    powerbi("Power BI")
  end
  subgraph gcp["Google Cloud Platform"]
    subgraph cloudamqp["CloudAMQP"]
      mqtt("MQTT broker")
    end
    subgraph kubernetes["Kubernetes Autopilot"]
      httppulsarpoller("GTFS Realtime Vehicle Position poller:<br/>http-pulsar-poller")
      mqttpulsarforwarder("Vehicle APC data MQTT forwarder:<br/>mqtt-pulsar-forwarder")
      pulsarclients("APC business logic<br/>as Pulsar clients")
      pulsarmqttforwardertopowerbi("Precise APC data MQTT forwarder:<br/>pulsar-mqtt-forwarder")
      pulsarmqttforwardertortpioutput("Anonymized APC data MQTT forwarder:<br/>pulsar-mqtt-forwarder")
      vehicleregistryapiuser("Vehicle registry poller:<br/>http-pulsar-poller")
    end
    subgraph streamnative["StreamNative Cloud Hosted"]
      pulsar("Apache Pulsar cluster")
    end
  end

  %% Edges

  httppulsarpoller --> pulsar
  mqtt --> mqttpulsarforwarder
  mqtt --> powerbi
  mqtt --> rtpioutput
  mqttpulsarforwarder --> pulsar
  onboard --> mqtt
  pulsar --> pulsarclients
  pulsar --> pulsarmqttforwardertopowerbi
  pulsar --> pulsarmqttforwardertortpioutput
  pulsarclients --> pulsar
  pulsarmqttforwardertopowerbi --> mqtt
  pulsarmqttforwardertortpioutput --> mqtt
  rtpiinput --> httppulsarpoller
  vehicleregistry --> vehicleregistryapiuser
  vehicleregistryapiuser --> pulsar

  %% Styling

  classDef done fill:#b2e08a,stroke:#000
  class httppulsarpoller done
  class mqtt done
  class mqttpulsarforwarder done
  class pulsar done
  class pulsarmqttforwardertopowerbi done
  class pulsarmqttforwardertortpioutput done
  class vehicleregistryapiuser done

  classDef env fill:#f7f7f7,stroke:#000
  class aws env
  class azure env
  class cloudamqp env
  class gcp env
  class kubernetes env
  class preset env
  class streamnative env

  classDef external fill:#1f78b6,stroke:#000,color:#fff
  class onboard external
  class powerbi external
  class rtpiinput external
  class rtpioutput external
  class vehicleregistry external

  classDef implemented fill:#dfc4f2,stroke:#000

  classDef todo fill:#fff,stroke:#ccc,stroke-width:1px,color:#666,stroke-dasharray: 6 6
  class pulsarclients todo

  %% Legend

  subgraph legend["Legend"]
    legendtodo("Todo")
    legendimplemented("Implemented but not deployed")
    legenddone("Done")
    legendexternal("External")
  end
  class legend env
  class legenddone done
  class legendexternal external
  class legendimplemented implemented
  class legendtodo todo
```

### Pulsar usage

```mermaid
flowchart TB

  %% Nodes

  anonymizer("anonymizer:<br/>waltti-apc-anonymizer")
  externalsinks("External sinks")
  externalsources("External sources")
  gtfsrtentitydeduplicatorfijyvaskyla("gtfsrt-vp-deduplicator-fi-jyvaskyla:<br/>pulsar-topic-deduplicator")
  gtfsrtentitydeduplicatorfikuopio("gtfsrt-vp-deduplicator-fi-kuopio:<br/>pulsar-topic-deduplicator")
  gtfsrtentityseparatorfijyvaskyla("gtfsrt-vp-entity-separator-fi-jyvaskyla:<br/>waltti-apc-gtfsrt-entity-separator")
  gtfsrtentityseparatorfikuopio("gtfsrt-vp-entity-separator-fi-kuopio:<br/>waltti-apc-gtfsrt-entity-separator")
  gtfsrtpollerfijyvaskyla("gtfsrt-vp-poller-fi-jyvaskyla:<br/>http-pulsar-poller (1 / AZ)")
  httppulsarpollerfikuopio("gtfsrt-vp-poller-fi-kuopio:<br/>http-pulsar-poller (1 / AZ)")
  matcher("journey-matcher:<br/>waltti-apc-journey-matcher")
  mqttapcmessagecleaner("vehicle-apc-message-cleaner:<br/>waltti-apc-mqtt-apc-message-cleaner")
  mqttdeduplicator("vehicle-apc-deduplicator:<br/>pulsar-topic-deduplicator")
  mqttpulsarforwarder("vehicle-apc-forwarder:<br/>mqtt-pulsar-forwarder (1 / AZ)")
  pulsarmqttforwardertopowerbi("Precise APC data MQTT forwarder:<br/>HSLdevcom/pulsar-mqtt-gateway")
  pulsarmqttforwardertortpioutput("Anonymized APC data MQTT forwarder:<br/>HSLdevcom/pulsar-mqtt-gateway")
  vehicleanonymizationprofiler("vehicle-anonymization-profiler:<br/>waltti-apc-vehicle-anonymization-profiler")
  vehicleregistrypollerfijyvaskyla("vehicle-registry-poller-fi-jyvaskyla:<br/>http-pulsar-poller (1 is enough)")
  vehicleregistrypollerfikuopio("vehicle-registry-poller-fi-kuopio:<br/>http-pulsar-poller (1 is enough)")
  subgraph nsaggregated["namespace aggregated (schema enforced)"]
    aggregated[/"aggregated-apc-journey"/]
  end
  subgraph nsanonymized["namespace anonymized (schema enforced)"]
    anonymized[/"anonymized-apc-journey"/]
  end
  subgraph nscleaned["namespace cleaned (schema enforced)"]
    mqttcleaned[/"mqtt-apc-from-vehicle-cleaned"/]
  end
  subgraph nsdeduplicated["namespace deduplicated"]
    gtfsrtdeduplicatedfijyvaskyla[/"gtfsrt-vp-deduplicated-fi-jyvaskyla"/]
    gtfsrtdeduplicatedfikuopio[/"gtfsrt-vp-deduplicated-fi-kuopio"/]
    mqttdeduplicated[/"mqtt-apc-from-vehicle-deduplicated"/]
  end
  subgraph nsprofiles["namespace profiles (schema enforced)"]
    vehicleanonymizationprofile[/"vehicle-anonymization-profiles"/]
  end
  subgraph nssource["namespace source"]
    gtfsrtrawfijyvaskyla[/"gtfsrt-vp-fi-jyvaskyla"/]
    gtfsrtrawfikuopio[/"gtfsrt-vp-fi-kuopio"/]
    gtfsrtseparatedfijyvaskyla[/"gtfsrt-vp-entities-separated-fi-jyvaskyla"/]
    gtfsrtseparatedfikuopio[/"gtfsrt-vp-entities-separated-fi-kuopio"/]
    mqttraw[/"mqtt-apc-from-vehicle"/]
    vehicledetailsfijyvaskyla[/"vehicle-catalogue-fi-jyvaskyla"/]
    vehicledetailsfikuopio[/"vehicle-catalogue-fi-kuopio"/]
  end

  %% Edges

  aggregated --> anonymizer
  aggregated --> pulsarmqttforwardertopowerbi
  anonymized --> pulsarmqttforwardertortpioutput
  anonymizer --> anonymized
  externalsources --> gtfsrtpollerfijyvaskyla
  externalsources --> httppulsarpollerfikuopio
  externalsources --> mqttpulsarforwarder
  externalsources --> vehicleregistrypollerfijyvaskyla
  externalsources --> vehicleregistrypollerfikuopio
  gtfsrtdeduplicatedfijyvaskyla --> matcher
  gtfsrtdeduplicatedfikuopio --> matcher
  gtfsrtentitydeduplicatorfijyvaskyla --> gtfsrtdeduplicatedfijyvaskyla
  gtfsrtentitydeduplicatorfikuopio --> gtfsrtdeduplicatedfikuopio
  gtfsrtentityseparatorfijyvaskyla --> gtfsrtseparatedfijyvaskyla
  gtfsrtentityseparatorfikuopio --> gtfsrtseparatedfikuopio
  gtfsrtrawfijyvaskyla --> gtfsrtentityseparatorfijyvaskyla
  gtfsrtrawfikuopio --> gtfsrtentityseparatorfikuopio
  gtfsrtseparatedfijyvaskyla --> gtfsrtentitydeduplicatorfijyvaskyla
  gtfsrtseparatedfikuopio --> gtfsrtentitydeduplicatorfikuopio
  gtfsrtpollerfijyvaskyla --> gtfsrtrawfijyvaskyla
  httppulsarpollerfikuopio --> gtfsrtrawfikuopio
  matcher --> aggregated
  mqttapcmessagecleaner --> mqttcleaned
  mqttcleaned --> matcher
  mqttdeduplicated --> mqttapcmessagecleaner
  mqttdeduplicator --> mqttdeduplicated
  mqttpulsarforwarder --> mqttraw
  mqttraw --> mqttdeduplicator
  pulsarmqttforwardertopowerbi --> externalsinks
  pulsarmqttforwardertortpioutput --> externalsinks
  vehicleanonymizationprofile --> anonymizer
  vehicleanonymizationprofiler --> vehicleanonymizationprofile
  vehicledetailsfijyvaskyla --> matcher
  vehicledetailsfikuopio --> matcher
  vehicledetailsfijyvaskyla --> vehicleanonymizationprofiler
  vehicledetailsfikuopio --> vehicleanonymizationprofiler
  vehicleregistrypollerfijyvaskyla --> vehicledetailsfijyvaskyla
  vehicleregistrypollerfikuopio --> vehicledetailsfikuopio

  %% Styling

  classDef done fill:#b2e08a,stroke:#000
  class aggregated done
  class anonymized done
  class anonymizer done
  class gtfsrtrawfijyvaskyla done
  class gtfsrtrawfikuopio done
  class gtfsrtpollerfijyvaskyla done
  class httppulsarpollerfikuopio done
  class matcher done
  class mqttdeduplicated done
  class mqttdeduplicator done
  class mqttpulsarforwarder done
  class mqttraw done
  class pulsarmqttforwardertopowerbi done
  class pulsarmqttforwardertortpioutput done
  class vehicledetailsfikuopio done
  class vehicleregistrypollerfikuopio done

  classDef env fill:#f7f7f7,stroke:#000
  class nsaggregated env
  class nscleaned env
  class nsdeduplicated env
  class nsanonymized env
  class nsprofiles env
  class nssource env

  classDef external fill:#1f78b6,stroke:#000,color:#fff
  class externalsinks external
  class externalsources external

  classDef implemented fill:#dfc4f2,stroke:#000
  class gtfsrtentitydeduplicatorfijyvaskyla implemented
  class gtfsrtentitydeduplicatorfikuopio implemented
  class vehicleanonymizationprofiler implemented
  class vehicleregistrypollerfijyvaskyla implemented

  classDef todo fill:#fff,stroke:#ccc,stroke-width:1px,color:#666,stroke-dasharray: 6 6
  class gtfsrtdeduplicatedfijyvaskyla todo
  class gtfsrtdeduplicatedfikuopio todo
  class gtfsrtentityseparatorfijyvaskyla todo
  class gtfsrtentityseparatorfikuopio todo
  class gtfsrtseparatedfijyvaskyla todo
  class gtfsrtseparatedfikuopio todo
  class mqttapcmessagecleaner todo
  class mqttcleaned todo
  class vehicleanonymizationprofile todo
  class vehicledetailsfijyvaskyla todo

  %% Legend

  subgraph legend["Legend"]
    legendtodo("Todo")
    legendimplemented("Implemented but not deployed")
    legenddone("Done")
    legendexternal("External")
  end
  class legend env
  class legenddone done
  class legendexternal external
  class legendimplemented implemented
  class legendtodo todo
```

## Other repositories

[http-pulsar-poller](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/http-pulsar-poller) polls an HTTP endpoint, e.g. a GTFS Realtime API, and sends the data into Apache Pulsar.

[mqtt-pulsar-forwarder](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/mqtt-pulsar-forwarder) forwards messages from the MQTT broker to the Apache Pulsar cluster.

[pulsar-mqtt-forwarder](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/pulsar-mqtt-forwarder) forwards messages from the Apache Pulsar cluster to the MQTT broker.

[pulsar-topic-deduplicator](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/pulsar-topic-deduplicator) forwards only the unique messages from Pulsar topics to another Pulsar topic.

[waltti-apc-anonymization-plan](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-anonymization-plan) describes the anonymization approach to be used for the GTFS Realtime API.

[waltti-apc-anonymizer](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-anonymizer) anonymizes accurate APC messages and outputs GTFS Realtime OccupancyStatus levels.

[waltti-apc-deployment](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-deployment) provisions, configures and deploys the services for the Waltti APC backend.

[waltti-apc-gcp-terraform](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-gcp-terraform) builds GCP infrastructure with Terraform.

[waltti-apc-journey-matcher](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-journey-matcher) matches APC messages from the vehicles with GTFS Realtime messages to augment the APC messages with vehicle journey metadata.

[waltti-apc-pilot-analysis](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-pilot-analysis) analyzes the pilot results.

[waltti-apc-pilot-spec](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-pilot-spec) contains the technical specifications for the pilot partners, i.e. the vendors who create the onboard counting devices.

[waltti-apc-vehicle-anonymization-profiler](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-vehicle-anonymization-profiler) creates anonymization profiles for vehicles.

[waltti-apc-vehicle-anonymization-profiler-library](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-vehicle-anonymization-profiler-library) is a fork of apc-anonymizer to have more control over changes. apc-anonymizer anonymizes automatic passenger counting (APC) results for public transit using differential privacy (DP).

[waltti-apc-vehicle-position-splitter](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-vehicle-position-splitter) splits Vehicle Position feed snapshots by vehicle and sends onwards only the data for vehicles with APC devices onboard, each vehicle in an individual message.

### Deprecated repositories

[waltti-apc-aggregation-test-data-generator](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-aggregation-test-data-generator) generates APC aggregation test data into Pulsar.

[waltti-apc-analytics-db-schema-migrator](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-analytics-db-schema-migrator) creates and maintains the Waltti APC analytics database schema.

[waltti-apc-analytics-db-sink](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-analytics-db-sink) forwards aggregated APC data into the analytics database.
