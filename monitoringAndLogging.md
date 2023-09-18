# Monitoring and Logging Plan for Waltti APC

## 1. Objective

To ensure effective monitoring and logging of the Waltti APC microservices, detect anomalies proactively, optimize performance, and ensure system health, while aligning with data anonymization and pilot specifications.

## 2. Cloud Monitoring Setup

### 2.1 Monitoring Microservices

Given the diversity of the microservices, monitor these primary metrics:

- **Performance Metrics:** CPU, Memory, and Network I/O for each microservice.
- **Application Health:** Uptime, response times, error rates.
- **Data Flow Metrics:** Message throughput for microservices like `http-pulsar-poller`, `mqtt-pulsar-forwarder`, `pulsar-mqtt-forwarder`, etc.
- **Database Operations:** For `waltti-apc-analytics-db-sink`, monitor DB write rates, errors, and latencies.

### 2.2 Set Up Alerting Policies on Slack

- **Microservice Failures:** Alerts for service unavailability or high error rates.
- **Resource Constraints:** Alerts for nearing resource limits in terms of CPU, memory, or storage.
- **Data Flow Anomalies:** Alerts for significant drops or spikes in data flow.

## 3. Cloud Logging Setup

### 3.1 Define Log Sources

Given the diversity and specificity of your repositories:

- **Operational Logs:** Logs from the microservices detailing their operations, errors, and state changes.
- **Data Processing Logs:** Logs from services responsible for data manipulation, such as `waltti-apc-anonymizer`, indicating successful operations, transformations, or any errors.
- **Infrastructure Logs:** Logs from `waltti-apc-deployment` and `waltti-apc-gcp-terraform` indicating the status of deployments and infrastructure changes.

### 3.2 Log Filters and Exclusions

- Focus on actionable logs; e.g., errors or warnings.
- Exclude verbose logs that don't have operational significance to manage costs.

### 3.3 Log-based Metrics

- Convert frequently occurring error logs into metrics to easily track and set alerts.

## 4. Special Considerations

### 4.1 Anonymization

As there's a dedicated service and plan (`waltti-apc-anonymization-plan` & `waltti-apc-anonymizer`) for data anonymization, it's crucial to ensure that no personally identifiable information (PII) gets logged. Regular audits and filters should be in place.

### 4.2 Integration with Slack

- Utilize GCP's integration capabilities to forward critical alerts to your team's Slack channel.

## 5. Documentation and Maintenance

- **Service Documentation:** Each microservice should have documentation detailing its logging and monitoring specifics.
- **Regular Reviews:** Periodically assess the effectiveness of the monitoring and logging setup. Adjust as the system evolves.
- **Feedback Loop:** Encourage team members to provide feedback on the usefulness of alerts and dashboards. Adjust to reduce noise and enhance utility.
