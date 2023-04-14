
## Who this guide is for
This guide will get you started with Instana on Kubernetes quickly. For production deployments you should read the complete documentation carefully and be aware of special cases that may apply to your specific Kubernetes configuration.

## Prerequisites
You will need access to a Kubernetes cluster (minimum version 1.21) with `kubectl` and `helm` (v3) configured on your local machine.

## Step 1: Install the agent
Follow the installation steps for helm.[IBM Documentation](https://www.ibm.com/docs/en/instana-observability/current?topic=requirements-installing-host-agent-kubernetes#install-by-using-the-helm-chart)

> **Note**
> While this  quickstart guide uses helm, we recommend using the [Instana Operator]() for production deployments.

## Step 2: Configure Applications
The AutoTrace WebHook can instrument many of your applications. For other applications, you will simply need to install a single Instana instrumentation library and initialize it.

_link to list of supported languages_

The instrumentation for some languages needs to know how to communicate with the agent. This step is required for the following languages:

* Node.js
* Go
* Ruby
* Python
* .NET Core

Add the following snippet to your pod deployment definitions:
```
spec:
  containers:
    env:
      - name: INSTANA_AGENT_HOST
        valueFrom:
          fieldRef:
            fieldPath: status.hostIP
```

You can see an example of this in the robot-shop application.

## Step 3: Configure Supported Technologies
Once the agent is installed and your applications are instrumented, any upstream or downstream services such as databases and messaging queues will be automatically discovered by the agent. For some technologies, specific configuration allows the Instana agent to collect additional data. You can check the list of supported technologies for any additional configuration steps based on the technologies that you have in use.

A very common configuration step is providing database credentials to the agent. For example, when monitoring PostgreSQL the Instana agent needs `SELECT` permissions on the `pg_stat_database` database.

https://www.ibm.com/docs/en/instana-observability/current?topic=technologies-monitoring-postgresql


## Enable the Agent Service (Optional)
The helm chart provides a number of helpful flags for commonly used configuration options: [Helm Chart Configuration Reference](https://github.com/instana/helm-charts/tree/main/instana-agent#configuration-reference). A common option is to enable the agent service which is a prerequisite for Prometheus, OpenTelemetry, and other agent APIs:

```
helm upgrade instana-agent \
--namespace instana-agent \
--reuse-values \
--set service.create=true
```

> **Note**
> We use the helm `--reuse-values` flag so that we don’t need to provide our API key or existing configuration when adding new configuration.  

> You may need to restart the agent pods after updating configuration.

## Enable OpenTelemetry Support (Optional)
The Instana agent can ingest metrics, traces, and logs* in OTLP format. This can be enabled for http, grpc, or both:

```
helm upgrade instana-agent \
--namespace instana-agent \
--reuse-values \
--set opentelemetry.grpc.enabled=true \
--set opentelemetry.http.enabled=true
```

### Configuring OpenTelemetry Services
Applications instrumented with OpenTelemetry need to be configured to send their telemetry to the Instana Agent’s endpoints.

```
spec:
  containers:
    env:
      - name: OTLP_TRACES_ENDPOINT
        valueFrom:
          fieldRef:
            fieldPath: status.hostIP
```

## Enable AutoTrace (Optional)
The Instana AuotTrace WebHook can automatically install instrumentation for Node.js, .NET Core, Ruby, and Python applications running in your cluster. For more information, see the complete AutoTrace documentation.

AutoTrace should not be used with applications that are instrumented with OpenTelemetry or other non-Instana instrumentation.

```
helm install --create-namespace --namespace instana-autotrace-webhook instana-autotrace-webhook \
  --repo https://agents.instana.io/helm instana-autotrace-webhook \
  --set webhook.imagePullCredentials.password=<download_key>
```

> **Note**
> You will need to replace `<download_key>` with your Instana API key.

## Additional Configuration Options
There are many configuration options 
[Host Agent Configuration Options](https://www.ibm.com/docs/en/instana-observability/current?topic=requirements-installing-host-agent-kubernetes#report-to-multiple-backends)
Updating ConfigMaps