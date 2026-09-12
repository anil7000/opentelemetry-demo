# Local telemetry troubleshooting checklist

Use this sequence when the webstore works but traces or dashboards are empty.
Keep application availability and telemetry delivery as separate checks.

## Match the Compose layers

The default stack combines core, full, observability and extras files.
Use the same environment and file order as [Makefile](Makefile):

```sh
docker compose --env-file .env --env-file .env.override \
  -f compose.yaml -f compose.full.yaml \
  -f compose.observability.yaml -f compose.extras.yaml config --quiet
```

Run from the repository root with Docker Compose v2 installed. This validates
configuration without starting containers. Do not publish a full rendered
configuration: environment interpolation can reveal credentials.

## Trace one synthetic request

1. Use the existing local webstore to make a synthetic request.
2. Record the request time and, if available, its trace ID.
3. Check whether the application emitted telemetry.
4. Check the Collector receiver, processor and exporter path.
5. Check the destination backend and its query time range.
6. Compare service names and resource attributes before blaming sampling.

| Observation | Investigate |
| --- | --- |
| UI is unreachable | Frontend proxy and application container health |
| UI works but no traces appear | Instrumentation endpoint and Collector pipeline |
| Only some services appear | Per-service exporter settings and resource names |
| Collector exporter errors | Backend availability, endpoint and protocol |
| Old traces appear but new ones do not | Time range, timestamps and current ingest health |

The default local interfaces documented in [CONTRIBUTING.md](CONTRIBUTING.md)
include the webstore at port 8080, Jaeger under /jaeger/ui/ and Grafana under
/grafana/. Do not expose this demo publicly with default settings.

Before changing a feature flag, record its previous value and use only synthetic
traffic. Restore it after the experiment. Do not clear all Docker data to debug
one service; broad pruning can affect unrelated projects.

## Development note

This troubleshooting guide was added with AI assistance. Upstream code,
licenses and contributor attribution remain unchanged.
