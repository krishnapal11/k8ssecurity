# Falco Custom Helm Chart

[Falco](https://falco.org) is a *Cloud Native Runtime Security* tool designed to detect anomalous activity in your applications. You can use Falco to monitor runtime security of your Kubernetes applications and internal components.

## Introduction

This Helm chart provides a custom configuration of Falco, emphasizing streamlined integration with Falcosidekick for comprehensive alert forwarding. The deployment of Falco in a Kubernetes cluster is managed through this **Helm chart**. This chart manages the lifecycle of Falco in a cluster by handling all the k8s objects needed by Falco to be seamlessly integrated in your environment. Based on the configuration in [values.yaml](./values.yaml) file, the chart will render and install the required k8s objects.

## Adding `falcosecurity` repository

Before installing the chart, add the `falcosecurity` charts repository:

```bash
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm repo update
```

## Installing the Chart

To install the chart with the release name `falco` in namespace `falco` run:

```bash
helm install falco falcosecurity/falco \
    --create-namespace \
    --namespace falco
```

After a few minutes Falco instances should be running on all your nodes.
> **Tip**: List Falco release using `helm list -n falco`, a release is a name used to track a specific deployment.

### Falco, Event Sources and Kubernetes
Falco can process events from various sources, including system calls (via drivers like eBPF or kernel modules) and plugins (like K8s Audit Logs). This chart focuses on a streamlined deployment, typically as a DaemonSet using the most suitable driver for your system, configurable via `values.yaml`.

### About Falco Artifacts
Falco rules and plugins (referred to as artifacts) can be managed using the `falcoctl` tool. This chart may use `falcoctl` to install and update these artifacts. For detailed configuration of `falcoctl`, refer to the `values.yaml` file.

### Deploying Falco in Kubernetes
This chart typically deploys Falco as a `DaemonSet` to ensure an instance runs on each node for comprehensive monitoring. The specific deployment kind (DaemonSet or Deployment) and its configuration are managed via the `values.yaml` file.

## Uninstalling the Chart

To uninstall a Falco release from your Kubernetes cluster always you helm. It will take care to remove all components deployed by the chart and clean up your environment. The following command will remove a release called `falco` in namespace `falco`;

```bash
helm uninstall falco --namespace falco
```

## Loading custom rules

Custom Falco rules can be added to your deployment. This is managed via the `customRules` key in the `values.yaml` file. You can define your rules there directly or reference ConfigMaps containing your rule definitions. For example:
```yaml
customRules:
  my_custom_rules.yaml: |-
    - rule: My Custom Rule
      desc: Detects a specific custom event
      condition: evt.type = my_custom_event
      output: "My custom event detected (fields: %evt.args)"
      priority: INFO
```
Refer to the `values.yaml` for more details on structuring custom rules.
## Deploy Falcosidekick with Falco

[`Falcosidekick`](https://github.com/falcosecurity/falcosidekick) is a companion tool for Falco, deployed as a sub-chart of this Falco Helm chart. It extends Falco's capabilities by forwarding alert events to a wide array of output channels. This allows for flexible integration into your existing security and operational workflows.

Some categories of supported integrations include (not an exhaustive list):
*   **Chat:** Slack, Microsoft Teams, Discord, Google Chat, Rocket.Chat, Mattermost, Telegram, Zoho Cliq, Webex.
*   **Logging & SIEM:** Elasticsearch, Loki, Splunk, Grafana Loki, AWS Security Lake, OpenSearch, Sumo Logic, Quickwit, ZincSearch, OpenObserve.
*   **Alerting & Incident Management:** PagerDuty, Opsgenie, Alertmanager, Grafana OnCall.
*   **Metrics & Observability:** Datadog, Prometheus, InfluxDB, StatsD, DogStatsD, Wavefront, Dynatrace, TimescaleDB, OTLP (traces and metrics).
*   **Generic Webhooks:** Send alerts to any compatible HTTP endpoint.
*   **Cloud Storage & Messaging:** AWS S3, AWS SNS, AWS SQS, AWS Kinesis, Azure Event Hubs, Google Cloud Pub/Sub, Google Cloud Storage, RabbitMQ, NATS, STAN (NATS Streaming), Kafka, Redis.
*   **Serverless Functions & Workflow Automation:** AWS Lambda, Google Cloud Functions, Google Cloud Run, Kubeless, OpenFaaS, Fission, Tekton, Node-RED, n8n.
*   **Policy & Compliance:** PolicyReport (for Kubernetes).
*   **Email:** SMTP.
*   **Other:** Gotify, Spyderbat, MQTT.

Falcosidekick can be installed and configured alongside `Falco` by setting `falcosidekick.enabled=true` in your `values.yaml` file or via Helm's `--set` flag. This main Falco chart automatically configures Falco to work with Falcosidekick when enabled.

All specific configurations for Falcosidekick and its numerous outputs are managed by prefixing the keys with `falcosidekick.` within the Falco chart's `values.yaml` (e.g., `falcosidekick.slack.webhookurl="YOUR_SLACK_WEBHOOK_URL"`).

For example, to enable the deployment of Falcosidekick and its optional UI, [`Falcosidekick-UI`](https://github.com/falcosecurity/falcosidekick-ui), you would add:
`--set falcosidekick.enabled=true --set falcosidekick.webui.enabled=true`
to your Helm install/upgrade command, or set these in your `values.yaml`.

If you use a Proxy in your cluster, the requests between `Falco` and `Falcosidekick` might be captured. To avoid this, use the full FQDN of `Falcosidekick` by setting `--set falcosidekick.fullfqdn=true`.

For a comprehensive list of all supported outputs and their detailed configuration parameters, please refer to the `values.yaml` file located within the Falcosidekick chart directory (`charts/falcosidekick/values.yaml`) and the official Falcosidekick documentation at its GitHub repository: [https://github.com/falcosecurity/falcosidekick](https://github.com/falcosecurity/falcosidekick).
