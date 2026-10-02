---
name: llm-traffic-governance
description: Load balances inference requests across LLM providers, manages token budgets, and implements circuit breaking.
license: Apache-2.0
---

# LLM Traffic Governance

## Overview
This skill governs outgoing LLM traffic, optimizing cost, latency, and reliability across model provider endpoints.

## Capabilities
- Routes requests based on real-time latency, queue depth, and provider token rates.
- Enforces tenant-level token budgets and dollar spending caps.
- Executes automated failover and backoff retries upon provider downtime.
