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
  subgraph aws["Amazon Web Services (eu-north-1, Stockholm)"]
    subgraph preset["Preset"]
      superset("Apache Superset")
    end
  end
  subgraph azure["Microsoft Azure"]
    powerbi("Power BI")
  end
  subgraph gcp["Google Cloud Platform (europe-west3, Frankfurt)"]
    db[("APC analytics PostgreSQL")]
    subgraph cloudamqp["CloudAMQP"]
      mqtt("MQTT broker")
    end
    subgraph kubernetes["Kubernetes Autopilot"]
      apitodb("APC analytics PostgreSQL feeder:<br/>waltti-apc-analytics-db-sink")
      apitopowerbi("API to Power BI")
      apitortpioutput("API to Waltti RTPI")
      dbschemamigrator("Database schema migrator:<br/>waltti-apc-analytics-db-schema-migrator")
      httppulsarpoller("GTFS Realtime forwarder:<br/>http-pulsar-poller")
      mqttpulsarforwarder("MQTT message forwarder:<br/>mqtt-pulsar-forwarder")
      pulsarclients("APC business logic<br/>as Pulsar clients")
      testdatagenerator("Test data generator:<br/>waltti-apc-aggregation-test-data-generator")
      vehicleregistryapiuser("Vehicle registry API user")
    end
    subgraph streamnative["StreamNative Cloud Hosted"]
      pulsar("Apache Pulsar cluster")
    end
  end

  %% Edges

  apitodb --> db
  apitopowerbi --> powerbi
  apitortpioutput --> rtpioutput
  db --> superset
  dbschemamigrator --> db
  httppulsarpoller --> pulsar
  mqtt --> mqttpulsarforwarder
  mqttpulsarforwarder --> pulsar
  onboard --> mqtt
  pulsar --> pulsarclients
  pulsarclients --> apitodb
  pulsarclients --> apitopowerbi
  pulsarclients --> apitortpioutput
  pulsarclients --> pulsar
  rtpiinput --> httppulsarpoller
  testdatagenerator --> pulsar
  vehicleregistry --> vehicleregistryapiuser
  vehicleregistryapiuser --> pulsar

  %% Styling

  classDef deprecated fill:#fbaf60,stroke:#000,stroke-width:1px,color:#000,stroke-dasharray: 6 6
  class apitodb deprecated
  class db deprecated
  class dbschemamigrator deprecated
  class superset deprecated
  class testdatagenerator deprecated

  classDef done fill:#b2e08a,stroke:#000
  class httppulsarpoller done
  class mqtt done
  class mqttpulsarforwarder done
  class pulsar done

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

  classDef todo fill:#fff,stroke:#ccc,stroke-width:1px,color:#666,stroke-dasharray: 6 6
  class apitopowerbi todo
  class apitortpioutput todo
  class pulsarclients todo
  class vehicleregistryapiuser todo

   %% Legend

  subgraph legend["Legend"]
    legendtodo("Todo")
    legenddone("Done")
    legenddeprecated("Deprecated")
    legendexternal("External")
  end
  class legend env
  class legenddeprecated deprecated
  class legenddone done
  class legendexternal external
  class legendtodo todo
```

### Pulsar usage

```mermaid
flowchart TB

  %% Nodes

  accumulator("waltti-apc-journey-accumulator")
  anonymizer("waltti-apc-anonymizer")
  apitopowerbi("API to Power BI")
  apitortpioutput("API to Waltti RTPI")
  dbsink("waltti-apc-analytics-db-sink")
  externalsinks("External sinks")
  externalsources("External sources")
  gtfsrtentitydeduplicatorfijyvaskyla("pulsar-topic-deduplicator")
  gtfsrtentitydeduplicatorfikuopio("pulsar-topic-deduplicator")
  gtfsrtentityseparatorfijyvaskyla("waltti-apc-gtfsrt-entity-separator")
  gtfsrtentityseparatorfikuopio("waltti-apc-gtfsrt-entity-separator")
  httppulsarpollerfijyvaskyla("http-pulsar-poller (1 / AZ)")
  httppulsarpollerfikuopio("http-pulsar-poller (1 / AZ)")
  matcher("waltti-apc-journey-matcher")
  mqttapcmessagecleaner("waltti-apc-mqtt-apc-message-cleaner")
  mqttdeduplicator("pulsar-topic-deduplicator")
  mqttpulsarforwarder("mqtt-pulsar-forwarder (1 / AZ)")
  vehicleregistryapiuser("Vehicle registry API user")
  subgraph nsaggregation["namespace aggregation"]
    accumulated[/"accumulated-apc-journey"/]
    aggregated[/"aggregated-apc-journey"/]
  end
  subgraph nscleaned["namespace cleaned"]
    mqttcleaned[/"mqtt-apc-from-vehicle-cleaned"/]
  end
  subgraph nsopen["namespace open"]
    anonymized[/"anonymized-apc-journey"/]
  end
  subgraph nsretained["namespace retained"]
    gtfsrtdeduplicatedfijyvaskyla[/"gtfsrt-vp-deduplicated-fi-jyvaskyla"/]
    gtfsrtdeduplicatedfikuopio[/"gtfsrt-vp-deduplicated-fi-kuopio"/]
    mqttdeduplicated[/"mqtt-apc-from-vehicle-deduplicated"/]
  end
  subgraph nssource["namespace source"]
    gtfsrtrawfijyvaskyla[/"gtfsrt-vp-fi-jyvaskyla"/]
    gtfsrtrawfikuopio[/"gtfsrt-vp-fi-kuopio"/]
    gtfsrtseparatedfijyvaskyla[/"gtfsrt-vp-entities-separated-fi-jyvaskyla"/]
    gtfsrtseparatedfikuopio[/"gtfsrt-vp-entities-separated-fi-kuopio"/]
    mqttraw[/"mqtt-apc-from-vehicle"/]
    vehicledetails[/"vehicle-details"/]
  end

  %% Edges

  accumulated --> anonymizer
  accumulator --> accumulated
  aggregated --> accumulator
  aggregated --> apitopowerbi
  aggregated --> dbsink
  anonymized --> apitortpioutput
  anonymizer --> anonymized
  apitopowerbi --> externalsinks
  apitortpioutput --> externalsinks
  dbsink --> externalsinks
  externalsources --> httppulsarpollerfijyvaskyla
  externalsources --> httppulsarpollerfikuopio
  externalsources --> mqttpulsarforwarder
  externalsources --> vehicleregistryapiuser
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
  httppulsarpollerfijyvaskyla --> gtfsrtrawfijyvaskyla
  httppulsarpollerfikuopio --> gtfsrtrawfikuopio
  matcher --> aggregated
  mqttapcmessagecleaner --> mqttcleaned
  mqttcleaned --> matcher
  mqttdeduplicated --> mqttapcmessagecleaner
  mqttdeduplicator --> mqttdeduplicated
  mqttpulsarforwarder --> mqttraw
  mqttraw --> mqttdeduplicator
  vehicledetails --> anonymizer
  vehicledetails --> matcher
  vehicleregistryapiuser --> vehicledetails

  %% Styling

  classDef deprecated fill:#fbaf60,stroke:#000,stroke-width:1px,color:#000,stroke-dasharray: 6 6
  class dbsink deprecated

  classDef done fill:#b2e08a,stroke:#000
  class aggregated done
  class gtfsrtentitydeduplicatorfijyvaskyla done
  class gtfsrtentitydeduplicatorfikuopio done
  class gtfsrtrawfijyvaskyla done
  class gtfsrtrawfikuopio done
  class httppulsarpollerfijyvaskyla done
  class httppulsarpollerfikuopio done
  class matcher done
  class mqttdeduplicated done
  class mqttdeduplicator done
  class mqttpulsarforwarder done
  class mqttraw done

  classDef env fill:#f7f7f7,stroke:#000
  class nsaggregation env
  class nscleaned env
  class nsopen env
  class nsretained env
  class nssource env

  classDef external fill:#1f78b6,stroke:#000,color:#fff
  class externalsinks external
  class externalsources external

  classDef todo fill:#fff,stroke:#ccc,stroke-width:1px,color:#666,stroke-dasharray: 6 6
  class accumulated todo
  class accumulator todo
  class anonymized todo
  class anonymizer todo
  class apitopowerbi todo
  class apitortpioutput todo
  class gtfsrtdeduplicatedfijyvaskyla todo
  class gtfsrtdeduplicatedfikuopio todo
  class gtfsrtentityseparatorfijyvaskyla todo
  class gtfsrtentityseparatorfikuopio todo
  class gtfsrtseparatedfijyvaskyla todo
  class gtfsrtseparatedfikuopio todo
  class mqttapcmessagecleaner todo
  class mqttcleaned todo
  class vehicledetails todo
  class vehicleregistryapiuser todo

  %% Legend

  subgraph legend["Legend"]
    legendtodo("Todo")
    legenddone("Done")
    legenddeprecated("Deprecated")
    legendexternal("External")
  end
  class legend env
  class legenddeprecated deprecated
  class legenddone done
  class legendexternal external
  class legendtodo todo
```

Later on, we might switch from a separate MQTT broker onto MQTT-on-Pulsar (MoP).
MoP seems like a fairly capable MQTT broker that persists every MQTT message onto Apache Pulsar automatically.

## Other repositories

[http-pulsar-poller](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/http-pulsar-poller) polls an HTTP endpoint, e.g. a GTFS Realtime API, and sends the data into Apache Pulsar.

[mqtt-pulsar-forwarder](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/mqtt-pulsar-forwarder) forwards messages from the MQTT broker to the Apache Pulsar cluster.

[pulsar-topic-deduplicator](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/pulsar-topic-deduplicator) forwards only the unique messages from Pulsar topics to another Pulsar topic.

[waltti-apc-aggregation-test-data-generator](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-aggregation-test-data-generator) generates APC aggregation test data into Pulsar.

[waltti-apc-analytics-db-schema-migrator](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-analytics-db-schema-migrator) creates and maintains the Waltti APC analytics database schema.

[waltti-apc-analytics-db-sink](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-analytics-db-sink) forwards aggregated APC data into the analytics database.

[waltti-apc-anonymization-plan](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-anonymization-plan) describes the anonymization approach to be used for the GTFS Realtime API.

[waltti-apc-deployment](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-deployment) provisions, configures and deploys the services for the Waltti APC backend.

[waltti-apc-gcp-terraform](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-gcp-terraform) builds GCP infrastructure with Terraform.

[waltti-apc-journey-matcher](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-journey-matcher) matches APC messages from the vehicles with GTFS Realtime messages to augment the APC messages with vehicle journey metadata.

[waltti-apc-pilot-analysis](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-pilot-analysis) analyzes the pilot results.

[waltti-apc-pilot-spec](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-pilot-spec) contains the technical specifications for the pilot partners, i.e. the vendors who create the onboard counting devices.
