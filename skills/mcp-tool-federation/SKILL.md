---
name: mcp-tool-federation
description: Discovers, federates, and multiplexes Model Context Protocol (MCP) tools across heterogeneous transport protocols.
license: Apache-2.0
---

# MCP Tool Federation

## Overview
This skill mediates tool discovery and invocation across external MCP servers via stdio, HTTP, Server-Sent Events (SSE), and Streamable HTTP.

## Capabilities
- Aggregates disparate tool schemas into a unified namespace catalog.
- Translates JSON-RPC 2.0 messages across transports with zero payload corruption.
- Handles upstream tool authorization, token injection, and health checks.
