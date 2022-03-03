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
      mqttpulsarforwarder("mqtt-pulsar-forwarder")
      httppulsarpoller("http-pulsar-poller")
      testdatagenerator("Test data generator<br/>as Pulsar client")
      pulsarclients("APC business logic<br/>as Pulsar clients")
      dbschemamigrator("Database schema migrator")
    end
    db[("APC analytics PostgreSQL")]
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

  %% Styling

  classDef env fill:#f7f7f7,stroke:#000
  class gcp env
  class kubernetes env
  class streamnative env
  class cloudamqp env
  class azure env

  classDef external fill:#1f78b6,stroke:#000,color:#fff
  class onboard external
  class rtpi external

  classDef done fill:#b2e08a,stroke:#000
  class mqtt done
  class pulsar done
  class mqttpulsarforwarder done

  classDef later fill:#fff,stroke:#ccc,stroke-width:1px,color:#666,stroke-dasharray: 6 6
  class httppulsarpoller later

  classDef next fill:#a5cee3,stroke:#000,stroke-width:1px,color:#000,stroke-dasharray: 6 6
  class testdatagenerator next
  class pulsarclients next
  class dbschemamigrator next
  class db next
  class powerbi next

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
    mqttdeduplicator("waltti-apc-mqtt-deduplicator")
    httppulsarpoller("http-pulsar-poller")
    gtfsrtdeduplicator("waltti-apc-gtfsrt-deduplicator")
    matcher("waltti-apc-journey-matcher")
    testdatagenerator("waltti-apc-test-data-generator")
    aggregator("waltti-apc-aggregator")
    dbarranger("waltti-apc-db-arranger")
    postgresqlfeeder("waltti-apc-db-feeder")
  end
  subgraph pulsartopics["Apache Pulsar topics"]
    subgraph nssource["namespace source"]
      mqttraw[/"mqtt-apc-from-vehicle"/]
      mqttdeduplicated[/"mqtt-apc-from-vehicle-deduplicated"/]
      gtfsrtrawjyvaskyla[/"gtfsrt-jyvaskyla"/]
      gtfsrtdedupjyvaskyla[/"gtfsrt-jyvaskyla-deduplicated"/]
    end
    subgraph nsmatch["namespace match"]
      matchedapcjourney[/"matched-apc-journey"/]
    end
    subgraph nsaggregation["namespace aggregation"]
      aggregated[/"aggregated-apc-journey"/]
    end
    subgraph nssink["namespace sink"]
      apcdb[/"apc-db"/]
    end
  end

  %% Edges

  mqttpulsarforwarder --> mqttraw
  mqttraw --> mqttdeduplicator
  mqttdeduplicator --> mqttdeduplicated
  httppulsarpoller --> gtfsrtrawjyvaskyla
  gtfsrtrawjyvaskyla --> gtfsrtdeduplicator
  gtfsrtdeduplicator --> gtfsrtdedupjyvaskyla
  mqttdeduplicated --> matcher
  gtfsrtdedupjyvaskyla --> matcher
  matcher --> matchedapcjourney
  testdatagenerator --> matchedapcjourney
  matchedapcjourney --> aggregator
  aggregator --> aggregated
  aggregated --> dbarranger
  dbarranger --> apcdb
  apcdb --> postgresqlfeeder

  %% Styling

  classDef env fill:#f7f7f7,stroke:#000
  class pulsartopics env
  class pulsarclients env
  class nssource env
  class nsmatch env
  class nsaggregation env
  class nssink env

  classDef done fill:#b2e08a,stroke:#000
  class mqttpulsarforwarder done
  class mqttraw done

  classDef later fill:#fff,stroke:#ccc,stroke-width:1px,color:#666,stroke-dasharray: 6 6
  class httppulsarpoller later
  class mqttdeduplicator later
  class gtfsrtdeduplicator later
  class mqttdeduplicated later
  class gtfsrtrawjyvaskyla later
  class gtfsrtdedupjyvaskyla later
  class matcher later

  classDef next fill:#a5cee3,stroke:#000,stroke-width:1px,color:#000,stroke-dasharray: 6 6

  class testdatagenerator next
  class aggregator next
  class dbarranger next
  class postgresqlfeeder next
  class matchedapcjourney next
  class aggregated next
  class apcdb next

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
