# waltti-apc

The automatic passenger counting (APC) pilot for Waltti.

This main repository documents the high-level structure of the project, links to the other repositories of the project and contains the issues and the Kanban board.

## Architecture

### High-level architecture

```mermaid
flowchart TB
  %% Nodes

  rtpi("Waltti RTPI")
  onboard("Onboard counting systems")
  subgraph cloudamqp["CloudAMQP"]
    mqtt("MQTT broker")
  end
  subgraph gcp["Google Cloud Platform"]
    subgraph streamnative["StreamNative Cloud Hosted"]
      pulsar("Apache Pulsar cluster")
    end
    subgraph kubernetes["Kubernetes Autopilot"]
      mqttpulsarforwarder("MQTT message forwarder:<br/>mqtt-pulsar-forwarder")
      httppulsarpoller("GTFS Realtime forwarder:<br/>http-pulsar-poller")
      testdatagenerator("Test data generator:<br/>waltti-apc-aggregation-test-data-generator")
      pulsarclients("APC business logic<br/>as Pulsar clients")
      dbschemamigrator("Database schema migrator:<br/>waltti-apc-analytics-db-schema-migrator")
    end
    db[("APC analytics PostgreSQL")]
  end
  subgraph preset ["preset.io"]
    superset("Apache Superset")
  end
  subgraph azure ["Microsoft Azure"]
    powerbi("Power BI")
  end

  %% Edges

  onboard --> mqtt
  mqtt --> mqttpulsarforwarder
  mqttpulsarforwarder --> pulsar
  rtpi --> httppulsarpoller
  httppulsarpoller --> pulsar
  testdatagenerator --> pulsar
  pulsarclients --> pulsar
  pulsar --> pulsarclients
  pulsarclients --> db
  dbschemamigrator --> db
  db --> powerbi
  db --> superset

  %% Styling

  classDef env fill:#f7f7f7,stroke:#000
  class gcp env
  class kubernetes env
  class streamnative env
  class cloudamqp env
  class azure env
  class preset env

  classDef external fill:#1f78b6,stroke:#000,color:#fff
  class onboard external
  class rtpi external
  class powerbi external

  classDef done fill:#b2e08a,stroke:#000
  class mqtt done
  class pulsar done
  class mqttpulsarforwarder done
  class superset done
  class dbschemamigrator done
  class testdatagenerator done
  class db done
  class httppulsarpoller done

  classDef later fill:#fff,stroke:#ccc,stroke-width:1px,color:#666,stroke-dasharray: 6 6

  classDef next fill:#a5cee3,stroke:#000,stroke-width:1px,color:#000,stroke-dasharray: 6 6
  class pulsarclients next

  %% Legend

  subgraph legend["Legend"]
    legendexternal("External")
    legenddone("Done")
    legendnext("Todo next")
    legendlater("Todo later")
  end
  class legend env
  class legendexternal external
  class legenddone done
  class legendnext next
  class legendlater later
```

### Pulsar usage

```mermaid
flowchart TB
  %% Nodes

  subgraph pulsarclients["Apache Pulsar clients"]
    mqttpulsarforwarder("mqtt-pulsar-forwarder")
    mqttdeduplicator("pulsar-topic-deduplicator")
    httppulsarpoller("http-pulsar-poller")
    matcher("waltti-apc-journey-matcher")
    testdatagenerator("waltti-apc-aggregation-test-data-generator")
    aggregator("waltti-apc-aggregator")
    dbsink("waltti-apc-analytics-db-sink")
  end
  subgraph pulsartopics["Apache Pulsar topics"]
    subgraph nssource["namespace source"]
      mqttraw[/"mqtt-apc-from-vehicle"/]
      mqttdeduplicated[/"mqtt-apc-from-vehicle-deduplicated"/]
      gtfsrtrawjyvaskyla[/"gtfs-realtime-vehicleposition-fi-jyvaskyla"/]
    end
    subgraph nsmatch["namespace match"]
      matchedapcjourney[/"matched-apc-journey"/]
    end
    subgraph nsaggregation["namespace aggregation"]
      aggregated[/"aggregated-apc-journey"/]
    end
  end

  %% Edges

  mqttpulsarforwarder --> mqttraw
  mqttraw --> mqttdeduplicator
  mqttdeduplicator --> mqttdeduplicated
  httppulsarpoller --> gtfsrtrawjyvaskyla
  mqttdeduplicated --> matcher
  gtfsrtrawjyvaskyla --> matcher
  matcher --> matchedapcjourney
  matchedapcjourney --> aggregator
  aggregator --> aggregated
  testdatagenerator --> aggregated
  aggregated --> dbsink

  %% Styling

  classDef env fill:#f7f7f7,stroke:#000
  class pulsartopics env
  class pulsarclients env
  class nssource env
  class nsmatch env
  class nsaggregation env

  classDef done fill:#b2e08a,stroke:#000
  class mqttpulsarforwarder done
  class mqttraw done
  class testdatagenerator done
  class dbsink done
  class aggregated done
  class httppulsarpoller done
  class mqttdeduplicator done
  class mqttdeduplicated done
  class gtfsrtrawjyvaskyla done

  classDef later fill:#fff,stroke:#ccc,stroke-width:1px,color:#666,stroke-dasharray: 6 6

  classDef next fill:#a5cee3,stroke:#000,stroke-width:1px,color:#000,stroke-dasharray: 6 6
  class aggregator next
  class matchedapcjourney next
  class matcher next

  %% Legend

  subgraph legend["Legend"]
    legenddone("Done")
    legendnext("Todo next")
    legendlater("Todo later")
  end
  class legend env
  class legenddone done
  class legendnext next
  class legendlater later
```

Later on, we might switch from a separate MQTT broker onto MQTT-on-Pulsar (MoP).
MoP seems like a fairly capable MQTT broker that persists every MQTT message onto Apache Pulsar automatically.

## Other repositories

[waltti-apc-pilot-spec](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-pilot-spec) contains the technical specifications for the pilot partners, i.e. the vendors who create the onboard counting devices.

[waltti-apc-deployment](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-deployment) provisions, configures and deploys the services for the Waltti APC backend.

[mqtt-pulsar-forwarder](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/mqtt-pulsar-forwarder) forwards messages from the MQTT broker to the Apache Pulsar cluster.

[waltti-apc-analytics-db-schema-migrator](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-analytics-db-schema-migrator) creates and maintains the Waltti APC analytics database schema.

[waltti-apc-aggregation-test-data-generator](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-aggregation-test-data-generator) generates APC aggregation test data into Pulsar.

[waltti-apc-analytics-db-sink](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-analytics-db-sink) forwards aggregated APC data into the analytics database.

[waltti-apc-anonymization-plan](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-anonymization-plan) describes the anonymization approach to be used for the GTFS Realtime API.
