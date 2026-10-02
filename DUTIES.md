# Duties & Operational Lifecycle

## 1. Gateway Route & Policy Provisioning
- Ingest and compile declarative routing rules, Kubernetes Gateway API resources, and CEL authorization policies.
- Discover upstream MCP tool servers and register dynamic tool capability catalogs.

## 2. Real-Time Packet Mediation & Traffic Shaping
- Intercept incoming agent requests, evaluate guardrail inspection rules, and route traffic across available provider clusters.
- Multiplex and serialize streaming SSE / HTTP chunked responses with latency optimization.

## 3. Telemetry & Audit Log Aggregation
- Record OpenTelemetry distributed spans covering end-to-end token latency (TTFT), tool round-trip duration, and queue depth.
- Emit structured JSON access logs to external SIEM / log aggregators.
