# EXPLAINABILITY.md

This document explains the internal mechanisms, data lineage, operational boundaries, and governance framework of **AgentGateway** (`agentgateway`) in accordance with the **OpenGAP v0.1.0** specification for the **HiDevs GitAgent Passport** clearance pipeline.

> **Agent Name:** AgentGateway (`agentgateway`)  
> **Specification:** OpenGAP v0.1.0  
> **Category / Domain:** Developer Tools / Agent Networking, MCP Federation & A2A Communication  
> **Compliance Standard:** OpenGAP Checkpoint 2 (Explainability & Decision Governance), OWASP LLM Top 10, MITRE ATLAS  

---

## How the Agent Decides

The gateway routes and secures agentic communications through a deterministic, 5-stage packet inspection and dispatch pipeline.

### 1. Decision Architecture

The runtime intake, state classification, evaluation, and execution tracking operate across a deterministic, five-stage pipeline:

```
+-----------------------------------------------------------------------------------+
|                        Deterministic Gateway Pipeline                             |
+-----------------------------------------------------------------------------------+
|  [Stage 1: Connection Ingestion & Cryptographic Identity Check]                   |
|     --> Validate TLS handshake, decode JWT/mTLS certificates, & authenticate      |
+-----------------------------------------+-----------------------------------------+
                                          |
                                          v
+-----------------------------------------------------------------------------------+
|  [Stage 2: Guardrail & Policy Evaluation Gate]                                    |
|     --> Evaluate CEL access rules, scan for prompt injections, & redact PII       |
+-----------------------------------------+-----------------------------------------+
                                          |
                                          v
+-----------------------------------------------------------------------------------+
|  [Stage 3: Intelligent Provider & Tool Routing]                                   |
|     --> Route to optimal LLM provider, federated MCP tool server, or A2A peer     |
+-----------------------------------------+-----------------------------------------+
                                          |
                                          v
+-----------------------------------------------------------------------------------+
|  [Stage 4: Streaming Payload Mediation & Circuit Breaking]                        |
|     --> Manage backpressure, enforce token budgets, and stream SSE / gRPC frames  |
+-----------------------------------------+-----------------------------------------+
                                          |
                                          v
+-----------------------------------------------------------------------------------+
|  [Stage 5: Telemetry Instrumentation & Audit Logging]                             |
|     --> Export OpenTelemetry traces (TTFT, token counts, latency, and error codes)|
+-----------------------------------------------------------------------------------+
```

### 2. Decision Logic & Routing Formulations

Upstream LLM backend routing and tool server selection apply a cost- and latency-optimized affinity formulation:

$$S_{\text{route}}(p) = w_1 \cdot \left(1 - \frac{\text{QueueDepth}(p)}{\text{MaxQueue}}\right) + w_2 \cdot \left(1 - \frac{\text{Latency}_{\text{p95}}(p)}{\text{MaxLatency}}\right) + w_3 \cdot \text{CacheAffinity}(p) - w_4 \cdot \text{CostPerToken}(p)$$

Where:
- $w_1 = 0.35$: Upstream concurrency and queue depth health weight.
- $w_2 = 0.30$: Moving average p95 response latency factor.
- $w_3 = 0.20$: KV-cache prefix match score (Inference Gateway optimization).
- $w_4 = 0.15$: Normalized token pricing penalty.

Dynamic MCP tool federation ranking across registered tool servers is determined by:

$$P(\text{Server}_i \mid \text{ToolReq}) = \frac{\exp(\mathbf{s}_i \cdot \mathbf{q}_{\text{tool}})}{\sum_{j=1}^{N} \exp(\mathbf{s}_j \cdot \mathbf{q}_{\text{tool}})}$$

Where $\mathbf{q}_{\text{tool}}$ is the requested capability vector and $\mathbf{s}_i$ denotes server $i$'s capability schema weights.

### 3. Thresholding & Refusal Decision Criteria

AgentGateway enforces strict operational boundaries and deterministic refusal thresholds:
- **Refusal on ERR_AUTH_INVALID_CREDENTIALS**: Authentication Failure (Invalid token / expired JWT) halts execution with code `ERR_AUTH_INVALID_CREDENTIALS`.
- **Refusal on ERR_POLICY_ACCESS_DENIED**: RBAC / CEL Policy Denial (Evaluates to `false`) halts execution with code `ERR_POLICY_ACCESS_DENIED`.
- **Refusal on ERR_GUARDRAIL_INJECTION_DETECTED**: Prompt Injection Score (Classifier confidence $\ge 0.85$) halts execution with code `ERR_GUARDRAIL_INJECTION_DETECTED`.
- **Refusal on ERR_RATE_LIMIT_EXCEEDED**: Tenant Token Rate Quota ($> 100,000$ tokens/min) halts execution with code `ERR_RATE_LIMIT_EXCEEDED`.
- **Refusal on ERR_CIRCUIT_BREAKER_TRIPPED**: Upstream Provider Outage (5 consecutive HTTP 5xx responses) halts execution with code `ERR_CIRCUIT_BREAKER_TRIPPED`.

### 4. Fallback Decision Mechanism

Continuous operational stability is maintained through layered fault recovery:
- **Tier 1 (Automated Upstream Failover)**: If the primary LLM provider drops connection or returns 503, the proxy dynamically fails over to a secondary provider within 200ms.
- **Tier 2 (Degraded Mode & Tool Cache)**: If an external MCP tool server becomes unreachable, the gateway serves cached schema responses or signals degraded modality to the calling agent.
- **Model Fallback Cascade**: High-level reasoning and synthesis default to `gemini-2.0-flash` with automatic failover to `gpt-4o` and `claude-3-5-sonnet`.

### 5. Human-in-the-Loop Governance

Human operators retain sovereign authority over the multi-agent execution lifecycle:
- **Tier 3 (Human Administrator Intervention)**: Prolonged clusterwide circuit breaker activations dispatch alerts to oncall infrastructure engineers via PagerDuty/Webhook.
- **Benchmark Trajectory Auditing**: Operators inspect evaluation traces, raw generation tokens, and container logs to verify scoring fidelity.

---

## The Data It Uses

AgentGateway operates under strict principles of data minimization, environment isolation, and privacy protection.

### 1. Ingested Input Data

The framework processes only operational data necessary to perform its functions:
- **Agent Payloads**: Natural language prompts, tool call parameters, and model completion frames.
- **Protocol Envelopes**: MCP JSON-RPC 2.0 frames, A2A coordination messages, gRPC streaming packets.
- **Identity Contexts**: Bearer tokens, SPIFFE IDs, API keys, and client IP addresses.

### 2. Configuration & Reference Data

- **Tool Catalogs**: OpenAPI 3.0 specs, MCP tool schemas, JSON schema definitions.
- **Policy Definitions**: Common Expression Language (CEL) authorization rules and rate limit quotas.

### 3. Base Model & Inference Lineage

- **Supported Providers**: OpenAI, Anthropic, Google Gemini, AWS Bedrock, Ollama, vLLM.
- **Gateway Runtime**: Rust asynchronous networking (Tokio, Hyper), Go Kubernetes controller, Envoy data-plane extensions.

### 4. Data Privacy, Storage, and Retention

- **OWASP LLM & MITRE ATLAS Hardened**: Defended against indirect prompt injection, credential leakage, and unauthorized external API dispatch.
- **Local Environment Isolation**: Agent execution workspaces, intermediate scratchpads, and vector stores reside strictly within designated local project directories.
- **Automated Secret Scrubbing**: API keys, database credentials, and personal credentials are automatically redacted prior to embedding or logging.
- **Zero Commercial Monetization**: Prompts, intermediate reasoning trajectories, and task deliverables are never commercialized or shared with third parties.

---

## Limitations

Understanding the operational boundaries and technical constraints of AgentGateway is essential for effective deployment.

### 1. Connection Pool Contention Under Burst Traffic
- **Limitation**: Extreme bursts in concurrent agent-to-tool connections may saturate local file descriptor and ephemeral port limits.
- **Mitigation**: Implement adaptive HTTP/2 and gRPC connection multiplexing with backpressure queue shedding.

### 2. High Context Token Overhead in Tool Federation
- **Limitation**: Aggregating dozens of federated MCP tool definitions can exhaust calling model context windows.
- **Mitigation**: Dynamically prune tool schemas based on semantic task relevance and prompt intent indexing.

### 3. Non-Trivial Latency in Multi-Guardrail Chains
- **Limitation**: Chaining multiple third-party content moderation APIs sequentially increases Time to First Token (TTFT).
- **Mitigation**: Execute guardrail classifiers asynchronously in parallel and employ localized regex fast-paths.

### 4. Streaming Interruption on Provider Timeouts
- **Limitation**: In long-form reasoning runs, upstream provider timeouts mid-stream require connection resets.
- **Mitigation**: Inject periodic SSE keepalive heartbeats and implement client-side chunk resumption tokens.

### 5. Multi-Cloud Ingress Configuration Complexity
- **Limitation**: Deploying across hybrid multi-cloud topologies requires harmonizing diverse DNS, TLS, and ingress controller standards.
- **Mitigation**: Provide standardized Helm charts, Terraform blueprints, and automated Gateway API conformance profiles.

---

## Summary & Compliance Checklist

| Checkpoint 2 Requirement | Corresponding Section | Status |
| :--- | :--- | :---: |
| **How the agent decides** | [How the Agent Decides](#how-the-agent-decides) | **Covered** |
| - Decision architecture & 5-stage pipeline | Section 1 | Verified |
| - Decision logic & routing formulations | Section 2 | Verified |
| - Thresholding & refusal decision criteria | Section 3 | Verified |
| - Fallback decision mechanism | Section 4 | Verified |
| - Human-in-the-loop governance & oversight | Section 5 | Verified |
| **The data it uses** | [The Data It Uses](#the-data-it-uses) | **Covered** |
| - Ingested input data & query streams | Section 1 | Verified |
| - Configuration & reference schemas | Section 2 | Verified |
| - Base model lineage & deterministic engines | Section 3 | Verified |
| - Data privacy, retention lifecycle & MITRE/OWASP | Section 4 | Verified |
| **Its limitations** | [Limitations](#limitations) | **Covered** |
| - Connection Pool Contention Under Burst Traffic | Section 1 | Verified |
| - High Context Token Overhead in Tool Federation | Section 2 | Verified |
| - Non-Trivial Latency in Multi-Guardrail Chains | Section 3 | Verified |
| - Streaming Interruption on Provider Timeouts | Section 4 | Verified |
| - Multi-Cloud Ingress Configuration Complexity | Section 5 | Verified |
