---
name: guardrail-policy-enforcement
description: Evaluates safety guardrails, scans for prompt injections, redacts sensitive data, and enforces CEL access policies.
license: Apache-2.0
---

# Guardrail Policy Enforcement

## Overview
This skill acts as a synchronous inspection layer filtering malicious prompts and securing corporate data boundaries.

## Capabilities
- Detects prompt injection and jailbreak patterns prior to upstream model forwarding.
- Sanitizes and masks Personally Identifiable Information (PII) and secret credentials.
- Evaluates Common Expression Language (CEL) policy expressions for fine-grained authorization.
