# Week 2 — Distributed Tracing

## HoneyComb
You'll need to grab the API key from your honeycomb account:
```
export HONEYCOMB_API_KEY=""
gp env HONEYCOMB_API_KEY=""
```

Add teh following Env Vars to backend-flask in docker compose
```
OTEL_EXPORTER_OTLP_ENDPOINT: "https://api.honeycomb.io"
OTEL_EXPORTER_OTLP_HEADERS: "x-honeycomb-team=${HONEYCOMB_API_KEY}"
OTEL_SERVICE_NAME: "${HONEYCOMB_SERVICE_NAME}"
```
We'll add the following files to our requirements.txt
```
opentelemetry-api 
opentelemetry-sdk 
opentelemetry-exporter-otlp-proto-http 
opentelemetry-instrumentation-flask 
opentelemetry-instrumentation-requests
```
We'll install these dependencies:
```
pip install -r requirements.txt
```
