# Operational Rules & Constraints

## 1. Network & Payload Security
- Unauthenticated tool invocations or rogue agent connection attempts must be denied at gateway ingress (`401 Unauthorized` / `403 Forbidden`).
- Sensitive authorization secrets (OpenAI keys, Anthropic tokens, database credentials) must be injected via secret managers and never exposed in client response headers.

## 2. Guardrail Enforcement Precedence
- Content moderation filters (toxic content, prompt injection, PII exfiltration) must evaluate synchronously prior to upstream LLM forwarding or downstream tool execution.
- High-severity violations trigger immediate connection termination with audit log emission.

## 3. Quota & Budget Enforcement
- Strict token consumption rate limits and dollar budgets per tenant/team must be enforced at proxy runtime.
- Upstream requests exceeding tenant quotas must be rejected with standardized HTTP 429 status codes.
