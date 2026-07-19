---
name: grafana-dashboards
description: Author Grafana dashboards with the JSON model, template variables, and provisioning. Use when creating, editing, or codifying dashboards.
---

# Grafana Dashboards

A dashboard is one JSON document: metadata, a `panels` array, `templating`
(variables), and `time` defaults. Author in the UI, then export or manage as
code.

## JSON model essentials

```json
{
  "title": "Service Overview",
  "uid": "svc-overview",
  "editable": true,
  "time": {"from": "now-6h", "to": "now"},
  "templating": {"list": [ ... ]},
  "panels": [
    {
      "type": "timeseries",
      "title": "Request rate",
      "gridPos": {"h": 8, "w": 12, "x": 0, "y": 0},
      "datasource": {"type": "prometheus", "uid": "${datasource}"},
      "targets": [
        {"refId": "A",
         "expr": "sum(rate(http_requests_total{job=\"$job\"}[$__rate_interval]))",
         "legendFormat": "{{status}}"}
      ],
      "fieldConfig": {"defaults": {"unit": "reqps"}, "overrides": []}
    }
  ]
}
```

- `uid` is the stable identity: keep it fixed across updates or links break.
- `gridPos` is a 24-column grid; `h` units are 30px rows.
- Common panel `type` values: `timeseries`, `stat`, `gauge`, `bargauge`,
  `table`, `logs`, `heatmap`, `piechart`, `row`.
- Each entry in `targets` is one query (`refId` A, B, C...); the shape of the
  target depends on the data source (`expr` for Prometheus/Loki).
- `fieldConfig.defaults` holds units, thresholds, min/max; `overrides` target
  specific series.

## Variables (templating)

Variable types: `query` (values from a data source), `custom` (static list),
`interval`, `datasource`, `textbox`, `constant`. Reference as `$var` or
`${var}` in queries, titles, and legends.

Prometheus value sources: `label_values(metric, label)` for a label dropdown.
Enable `includeAll` and `multi` for multi-select, and remember queries must
then use regex matching: `{job=~"$job"}`.

Built-ins: `$__rate_interval` (preferred inside `rate()`), `$__interval`,
`$__range`, `$from`/`$to`.

## Provisioning (dashboards as code)

File provisioning: drop a provider YAML in
`conf/provisioning/dashboards/` and dashboard JSON files in the target path:

```yaml
apiVersion: 1
providers:
  - name: default
    folder: Services
    type: file
    options:
      path: /var/lib/grafana/dashboards
      foldersFromFilesStructure: true
```

Provisioned dashboards are read-only in the UI by default (set
`allowUiUpdates: true` to relax). Alternatives: Terraform
(`grafana_dashboard` resource) or the HTTP API:

```
POST /api/dashboards/db
{"dashboard": { ...json, "id": null }, "folderUid": "...", "overwrite": true}
```

Auth for the API uses a service account token (see `grafana-setup`).

## Practices

- One dashboard per service, rows per concern (traffic, latency, errors,
  saturation); avoid 50-panel walls.
- Parameterize the data source with a `datasource` variable so one JSON works
  across environments.
- Strip environment-specific `id` (set null) when exporting for reuse; keep
  `uid` deliberate.
- The bundled mcp-grafana server can search dashboards, fetch panel queries,
  and update dashboards directly on a live instance.
