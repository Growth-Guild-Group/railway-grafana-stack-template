# Grafana Stack on Railway

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/template/8TLSQD?referralCode=IFlm92)

## What is this template

This template deploys a complete Grafana observability stack on Railway with just one click! The stack includes four integrated services:

- **Grafana**: The leading open-source analytics and monitoring solution
- **Loki**: A horizontally-scalable, highly-available log aggregation system
- **Prometheus**: A powerful metrics collection and alerting system
- **Tempo**: A high-scale distributed tracing backend

This template is perfect for teams who need a comprehensive observability solution for their railway project without the hassle of manual configuration and infrastructure management.

### Key Features

- **Pre-configured Integration**: _All services come pre-connected_, so Grafana is ready to query your data immediately.
- **Persistent Storage**: All four services use Railway volumes to ensure your data, dashboards, and configurations persist between updates and deploys.
- **Version Control**: Pin specific Docker image versions for each service using environment variables.
- **Customizable**: Fork the repository to customize configuration files for any service. You can take full control and edit anything you'd need to as you scale.
- **One-Click Deploy**: Get a complete Grafana-based observability stack running in minutes.

## Quick Start Guide

1. Click the "Deploy on Railway" button at the top of this page
2. Enter your desired Grafana admin username in the `GF_SECURITY_ADMIN_USER` variable
3. Leave all other variables at their defaults (or customize as needed)
4. Wait for your stack to deploy (this typically takes 3-5 minutes)
5. Navigate to the Grafana URL provided by Railway
6. Log in with your admin username and the auto-generated password found in the `GF_SECURITY_ADMIN_PASSWORD` environment variable
7. Hook up your applications to the datasources.
8. Create dashboards, alerts, and explore your data in Grafana!

## Optional Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `GF_SECURITY_ADMIN_USER` | Username for the Grafana admin account | Required input |
| `GF_SECURITY_ADMIN_PASSWORD` | Password for the Grafana admin account | Auto-generated secure string |
| `GF_DEFAULT_INSTANCE_NAME` | Name of your Grafana instance | `Grafana on Railway` |
| `GF_INSTALL_PLUGINS` | Comma-separated list of Grafana plugins to install | `grafana-simple-json-datasource,grafana-piechart-panel,grafana-worldmap-panel,grafana-clock-panel` |

### Internal Service URLs

The Grafana service exposes these environment variables that you can reference in your other Railway applications to easily send data to your observability stack:

| Variable | Description | Usage |
|----------|-------------|-------|
| `LOKI_INTERNAL_URL` | Internal URL for the Loki service | Use in your applications to send logs to and query Loki |
| `PROMETHEUS_INTERNAL_URL` | Internal URL for the Prometheus service | Use in your applications to send metrics to and query Prometheus |
| `TEMPO_INTERNAL_URL` | Internal URL for the Tempo service | Use in your applications to query Tempo |

These variables make it easy to configure your other Railway services to send telemetry data to your observability stack.

Tempo also exposes a few variables to make it easier to push tracing information to the service using either HTTP or GRPC

| Variable | Description | Usage |
|----------|-------------|-------|
| `INTERNAL_HTTP_INGEST` | Internal HTTP ingest server URL for Tempo | Use in your applications to send traces to tempo via HTTP |
| `INTERNAL_GRPC_INGEST` | Internal GRPC ingest server URL for Tempo | Use in your applications to send traces to tempo via GRPC |

### Version Control

Each service has its own `VERSION` environment variable that can be set independently in each service's settings in the Railway dashboard:

- **Grafana Service**: Set `VERSION` to control the Grafana Docker image tag
- **Loki Service**: Set `VERSION` to control the Loki Docker image tag
- **Prometheus Service**: Set `VERSION` to control the Prometheus Docker image tag
- **Tempo Service**: Set `VERSION` to control the Tempo Docker image tag

By default, all services use the `latest` tag, but you can pin specific versions for stability:

Examples:
- Grafana: `VERSION=11.5.2`
- Loki: `VERSION=3.4.2`
- Prometheus: `VERSION=v3.2.1`
- Tempo: `VERSION=2.9.0`

This allows you to update each component independently as needed.

> **⚠️ Note on Tempo v2.10.0**: There is a known issue with Tempo v2.10.0 where the `compactor` configuration block is not recognized, causing startup failures. This template is pinned to v2.9.0 until this issue is resolved in a future release.

## Project Structure & Services

This template deploys four interconnected services:

### Grafana
- The central visualization and dashboarding platform
- Pre-configured with connections to all other services
- Persistent volume for storing dashboards, users, and configurations
- Comes with useful plugins pre-installed
- Exposes internal URLs for other Railway services to connect to Loki, Prometheus, and Tempo

### Prometheus
- Time-series database for metrics collection
- Configured with sensible defaults for monitoring
- Persistent volume for metrics data

### Loki
- Log aggregation system designed to be cost-effective
- Horizontally scalable architecture
- Persistent volume for log storage

### Tempo
- Distributed tracing system for tracking requests across services
- High-performance trace storage
- Persistent volume for trace data

All services are deployed using official Docker images and configured to work together seamlessly.

## Connecting Your Applications

### Using [Locomotive](https://railway.com/template/jP9r-f) for Loki

You can easily ingest *all* of your railway logs into Loki from *any* service using [Locomotive](https://railway.com/template/jP9r-f). Just spin up their template, drop in your Railway API key, the ID of the services you want to monitor, and a link to your new Loki instance and logs will start flowing! no code changes needed anywhere!

### Using OpenTelemetry libraries for Tempo 

Tempo is a bit different than both Prometheus and Loki in that exposes separate GRPC and HTTP servers on ports `:4317` and `:4318` respectively specifically for ingesting your tracing data or "spans".

When configuring your application to send traces to Tempo, please use one of the preconfigured variables in the Tempo service: `INTERNAL_HTTP_INGEST` or `INTERNAL_GRPC_INGEST`.

Another thing to note is that the ingest API endpoint for the HTTP server is `/v1/traces`. For a working example of this in a node.js express API, see `/examples/api/tracer.js` in our GitHub repository.

### Using otherwise standard observability tooling

To send data from your other Railway applications to this observability stack:

1. In your application's Railway service, add environment variables that reference the internal URLs:
   ```
   LOKI_URL=${{Grafana.LOKI_INTERNAL_URL}}
   PROMETHEUS_URL=${{Grafana.PROMETHEUS_INTERNAL_URL}}
   TEMPO_URL=${{Grafana.TEMPO_INTERNAL_URL}}
   ```
2. Configure your application's logging, metrics, or tracing libraries to use these URLs
3. Your application data will automatically appear in your Grafana dashboards

## Customizing Your Stack

To customize the configuration of Loki, Prometheus, or Tempo:

1. Fork the [GitHub repository](https://github.com/yourusername/grafana-railway-template)
2. Modify the configuration files in their respective directories
3. In Railway, disconnect the service you want to customize
4. Reconnect the service to your forked repository
5. Deploy the updated service

The pre-configured Grafana connections will continue to work with your customized services.

## lucidserum: required config & gotchas

This fork backs lucidserum's error-observability pipeline (apps → **Alloy** → this stack). A few
settings are load-bearing and easy to get subtly wrong — set them explicitly.

### Grafana service env vars

| Variable | Value | Why |
|----------|-------|-----|
| `GF_SERVER_ROOT_URL` | `https://${{RAILWAY_PUBLIC_DOMAIN}}` | Without it Grafana defaults to `http://localhost:3000`, so **every alert-notification link (Discord `Source:`/`Silence:`) points at localhost**. |
| `PROMETHEUS_INTERNAL_URL` | `http://prometheus.railway.internal:9090` | Prometheus datasource + alert-rule query target. **Use the bare service host** — Railway private domains dropped the old `-suffix` alias; a stale value 502s the datasource and fires a `DatasourceError`. |
| `LOKI_INTERNAL_URL` | `http://loki.railway.internal:3100` | Loki query datasource. |
| `TEMPO_INTERNAL_URL` | `http://tempo.railway.internal:3200` | Tempo **query** API — `:3200`, NOT the `:4317`/`:4318` OTLP ingest ports. |
| `DISCORD_WEBHOOK_URL` | *(your webhook)* | Alert contact point (`grafana/alerting/contactpoints.yaml`). |

### Alloy collector env vars (the OTLP ingress — separate service)

| Variable | Value | Gotcha |
|----------|-------|--------|
| `PROMETHEUS_REMOTE_WRITE_URL` | `http://prometheus.railway.internal:9090/api/v1/write` | Prometheus needs `--web.enable-remote-write-receiver`. |
| `LOKI_PUSH_URL` | `http://loki.railway.internal:3100/loki/api/v1/push` | **Port `:3100`, NOT `:3000`** — `:3000` is Grafana's port; pushing there is `connection refused` and logs are silently dropped. |
| `TEMPO_OTLP_ENDPOINT` | `tempo.railway.internal:4317` | **Bare `host:port`, no scheme** — a `grpc://`/`http://` prefix → `too many colons in address` and spans drop. |

### Notes

- **Alerting is Grafana-managed only** — rules live in `grafana/alerting/`. The datasources set
  `jsonData.manageAlerts: false` so the Alerting UI doesn't query a (nonexistent) Loki/Prometheus
  ruler; otherwise Loki 404s with *"Cannot load rules for this datasource."*
- **Datasource cross-links use the explicit `correlations:` key**, never jsonData
  `derivedFields`/`tracesToLogsV2`/`exemplarTraceIdDestinations` — the latter migrate inline during
  provisioning and can poison the Grafana volume ("data source not found" on every boot).
- **Verify data-plane legs at the sink, not at boot** — an exporter that logs no error at startup
  isn't proven; it only errors when it actually pushes. Query Loki/Tempo/Prometheus for real records.

## Synthetic uptime monitoring — Layer A (engineering) + the Layer B plan

**Layer A — internal synthetic monitoring** (this repo). A **Blackbox Exporter** service (`blackbox/`,
listens on `:9115`) that Prometheus scrapes with the multi-target pattern: `prometheus/prom.yml` calls
`/probe?target=<url>&module=http_2xx` per target, blackbox performs the HTTP probe, and per-target
labels (`service`/`environment`/`surface`/`criticality`/`provider`) travel with the `probe_*` series.

- **Deploy**: new Railway service in this project+environment, Root Directory = `blackbox` (Dockerfile
  builder, driven by `blackbox/railway.json`). No env/secrets — it probes public URLs. Internal host
  `blackbox.railway.internal:9115` must match the scrape config's `replacement`.
- **Targets**: edit the `blackbox` job's `static_configs` in `prometheus/prom.yml` — one entry per
  target, each with its own labels. Starts with `https://dev-api.lucidserum.com/health`.
- **Alerts** (`grafana/alerting/rules.yaml`, group `synthetic-monitoring`, → Discord, all
  `audience: engineering`): `synthetic-probe-down` (a target failed), `synthetic-blackbox-exporter-down`
  (the probe system itself is blind — kept **distinct** from a target failure), and
  `synthetic-tls-cert-expiring` (< 14d).
- **Dashboard**: `grafana/dashboards/synthetic-uptime.json` (probe up/down, latency, days-to-cert-expiry,
  exporter health).

> ⚠️ **Layer A runs INSIDE the Railway/observability failure domain** — a total Railway/provider/DNS
> outage takes it (and Grafana) down too. It is **engineering observability only**: it **must not** be
> wired to a customer-facing status page, and dev probes must never surface publicly. The
> `audience: engineering` label marks that boundary.

**Layer B — independent external synthetic monitoring** (NOT in this repo; a **required pre-launch
follow-up**). A probe hosted OUTSIDE Railway (e.g. Better Stack / Instatus / a self-hosted Gatus on a
different provider) hitting the same public URLs. It is the out-of-domain "is the provider / DNS /
Railway itself up" backstop and the **only** acceptable source for a public uptime/status page.

**Not covered by blackbox**: `worker` / `agent` have no public HTTP surface — their liveness belongs to
**heartbeat / metric-freshness / job-backlog** alerts off the metrics they already push (e.g. stale
`target_info`, rising outbox backlog), not an HTTP probe.

## Additional Resources

- [Locomotive: a loki transport for railway services](https://railway.com/template/jP9r-f)
- [Grafana Documentation](https://grafana.com/docs/grafana/latest/)
- [Loki Documentation](https://grafana.com/docs/loki/latest/)
- [Prometheus Documentation](https://prometheus.io/docs/introduction/overview/)
- [Tempo Documentation](https://grafana.com/docs/tempo/latest/)
- [Grafana Community Forums](https://community.grafana.com/)
- [Grafana Plugins Directory](https://grafana.com/grafana/plugins/)

---

Developed and maintained by [Mykal](https://mykal.codes). For issues or suggestions, please open an issue on the [GitHub repository](https://github.com/MykalMachon/grafana-stack-railway).
