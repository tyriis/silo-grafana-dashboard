# Silo / MinIO Metrics v3 Grafana Dashboard

A Grafana dashboard for S3-compatible object storage that exposes the MinIO
Metrics v3 API, derived from
[FedericoAntoniazzi/minio-grafana-dashboard-metrics-v3](https://github.com/FedericoAntoniazzi/minio-grafana-dashboard-metrics-v3)
and retargeted at [Silo](https://github.com/pgsty/silo), the maintained MinIO
fork.

This repository carries one dashboard. The upstream file was replaced by the
derivative rather than kept beside it, so `dashboard.json` here is **not** the
original: it is the 13-panel Silo variant, and `NOTICE` lists every change.

## Files

| File | What it is |
|---|---|
| `dashboard.json` | The Silo variant: 13 panels, VictoriaMetrics-friendly, `host`-aware. This is the file to import. |
| `NOTICE` | Attribution and the full list of modifications (Apache-2.0 §4(b)). |
| `LICENSE` | Apache License 2.0, as distributed with the original work. |

## Attribution

The original work is **"MinIO Metrics v3 Grafana Dashboard"** by
**Federico Antoniazzi**, with contributions from **Adam Charnock**:

- Source: https://github.com/FedericoAntoniazzi/minio-grafana-dashboard-metrics-v3
- Fork point: `main` @ `5ba0fb4072b2fecbabff2cedfdcb9cb58f9798a4`
- Licence: Apache-2.0

That dashboard is itself based on [MinIO Dashboard (#13502)](https://grafana.com/grafana/dashboards/13502-minio-dashboard/).
This repository is a derivative work; see `NOTICE` for every change made, and
`LICENSE` for the terms. Not affiliated with or endorsed by MinIO, Inc.

## What it targets

`dashboard.json` targets a Silo (or MinIO) server scraped on the metrics
v3 root path, one scrape job per instance, with the instance identified by a
`host` label:

```yaml
- job_name: minio
  metrics_path: /minio/metrics/v3
  static_configs:
    - targets: ["127.0.0.1:9000"]
  bearer_token: "<jwt>"
```

What differs from the upstream dashboard, in short: 13 panels instead of 37,
no KMS / erasure-set / healing / internode / scanner-counter panels, per-bucket
usage fixed to `sum` instead of `max`, zero-elided series held at 0 with
`or vector(0)`, the scanner panel repointed at `minio_scanner_last_activity_seconds`,
queries grouped `by (host)` so two single-node instances never double count, and
the data source switched to `victoriametrics`. The full list is in `NOTICE`.

Requirements:

- Metrics v3 enabled: `GET /minio/metrics/v3` (the token needs the
  `admin:Prometheus` action; the built-in `readonly` policy is not enough).
- A `host` label on the scraped series — an external label on the scrape job is
  the simplest way.
- A Prometheus-compatible data source that holds those series.

## Install

### Grafana UI

Import `dashboard.json`, then map `DS_PROMETHEUS` to your data source.

### grafana-operator

```yaml
apiVersion: grafana.integreatly.org/v1beta1
kind: GrafanaDashboard
metadata:
  name: silo
spec:
  instanceSelector:
    matchLabels:
      grafana.internal/instance: grafana
  datasources:
    - datasourceName: victoriametrics
      inputName: DS_PROMETHEUS
  url: https://raw.githubusercontent.com/tyriis/silo-grafana-dashboard/COMMIT_SHA/dashboard.json
```

Pin `COMMIT_SHA` rather than `main`, so what runs in the cluster matches a
reviewable commit.

### Helm

```yaml
dashboards:
  silo:
    url: "https://raw.githubusercontent.com/tyriis/silo-grafana-dashboard/COMMIT_SHA/dashboard.json"
    datasource:
      - name: DS_PROMETHEUS
        value: 'YOUR-DATA-SOURCE-NAME'
```

## Upstream README

The sections below are inherited from the original project.

### Why This Project Exists

MinIO has deprecated their Metrics v2 API and shifted focus to their embedded
dashboard solution. While this works for many people, a Grafana dashboard is
preferable when you want integration with an existing monitoring stack,
customisation beyond the embedded dashboard, one view across systems, or
alerting (see [minio/minio#20678](https://github.com/minio/minio/issues/20678)).

### Features (upstream)

- Updated for Metrics v3
- Based on the widely-used MinIO Dashboard (#13502)
- Drop-in replacement for dashboards built on Metrics v2
- Monitors throughput, latency, error rates, capacity, node health and S3 API use

### Migrating from Metrics v2

- Metric names changed; this dashboard accounts for that
- Some metrics were consolidated or restructured
- Metrics v3 adds series that v2 never had

### Limitations

- Community-maintained, not supported by MinIO
- Some features of the embedded dashboard are not replicated
- Requires updating when MinIO changes its metrics format
- Metrics v3 elides zero values, so panels need `or vector(0)`; a panel that
  silently blanks is usually a healthy-but-idle series, not a broken query

### Contributing

Pull requests are welcome, especially for metric-name drift as Silo and MinIO
evolve.

### Acknowledgements

- The MinIO team, for the object storage server
- Contributors to [MinIO Dashboard (#13502)](https://grafana.com/grafana/dashboards/13502-minio-dashboard/)
- Federico Antoniazzi and Adam Charnock, for the v3 port this repository builds on

### Disclaimer

Not affiliated with or endorsed by MinIO, Inc. MinIO is a trademark of MinIO, Inc.
