---
name: "mcp-builder"
displayName: "MCP Server Builder"
description: "Guide for creating high-quality MCP servers that enable LLMs to interact with external services. Covers TypeScript and Python implementations with best practices, tool design, and evaluation creation."
keywords: ["mcp", "model-context-protocol", "mcp-server", "fastmcp", "typescript-sdk"]
author: "Anthropic"
---

# MCP Server Development Guide

## Overview

Create MCP (Model Context Protocol) servers that enable LLMs to interact with external services through well-designed tools. The quality of an MCP server is measured by how well it enables LLMs to accomplish real-world tasks.

## Available Steering Files

- **best-practices** - Core MCP guidelines: naming, responses, pagination, transport, security
- **typescript-guide** - Complete TypeScript/Node implementation guide with Zod, project structure, examples
- **python-guide** - Complete Python/FastMCP implementation guide with Pydantic, examples
- **evaluation-guide** - Creating comprehensive evaluations to test MCP server effectiveness

## High-Level Workflow

### Phase 1: Deep Research and Planning

**API Coverage vs. Workflow Tools:**
Balance comprehensive API endpoint coverage with specialized workflow tools. When uncertain, prioritize comprehensive API coverage.

**Tool Naming and Discoverability:**
Use consistent prefixes (e.g., `github_create_issue`, `github_list_repos`) and action-oriented naming.

**Study MCP Protocol Documentation:**
Start with the sitemap: `https://modelcontextprotocol.io/sitemap.xml`
Fetch specific pages with `.md` suffix for markdown format.

**Recommended stack:**
- TypeScript (recommended) with Zod for validation
- Python with FastMCP and Pydantic
- Streamable HTTP for remote servers, stdio for local servers

**Plan Your Implementation:**
Review the service's API documentation. Prioritize comprehensive API coverage. List endpoints to implement, starting with the most common operations.

### Phase 2: Implementation

1. Set up project structure (see language-specific steering files)
2. Implement core infrastructure: API client, error handling, response formatting, pagination
3. Implement tools with:
   - Input schemas (Zod or Pydantic)
   - Output schemas where possible
   - Clear tool descriptions
   - Async/await for I/O
   - Proper error handling with actionable messages
   - Annotations: `readOnlyHint`, `destructiveHint`, `idempotentHint`, `openWorldHint`

### Phase 3: Review and Test

- No duplicated code (DRY principle)
- Consistent error handling
- Full type coverage
- Clear tool descriptions
- Build and test with MCP Inspector: `npx @modelcontextprotocol/inspector`

### Phase 4: Create Evaluations

Create 10 complex, realistic evaluation questions that test whether LLMs can effectively use your MCP server. See the evaluation-guide steering file for details.

## Best Practices Summary

- Use snake_case tool names with service prefix: `slack_send_message`
- Support both JSON and Markdown response formats
- Implement pagination with `has_more`, `next_offset`, `total_count`
- Store API keys in environment variables, never in code
- Provide actionable error messages that guide toward solutions
- Extract common functionality into reusable helpers
- Use strict type validation (Zod `.strict()` or Pydantic `extra='forbid'`)

## License & Attribution

**Original Work:** Converted from [mcp-builder](https://github.com/anthropics/skills/tree/main/skills/mcp-builder) by Anthropic.
**License:** See original repository for complete terms.
