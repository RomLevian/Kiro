# MCP Server Best Practices

## Quick Reference

### Server Naming
- **Python**: `{service}_mcp` (e.g., `slack_mcp`)
- **Node/TypeScript**: `{service}-mcp-server` (e.g., `slack-mcp-server`)

### Tool Naming
- Use snake_case with service prefix
- Format: `{service}_{action}_{resource}`
- Example: `slack_send_message`, `github_create_issue`

### Response Formats
- Support both JSON and Markdown formats
- JSON for programmatic processing
- Markdown for human readability

### Pagination
- Always respect `limit` parameter
- Return `has_more`, `next_offset`, `total_count`
- Default to 20-50 items

### Transport
- **Streamable HTTP**: For remote servers, multi-client scenarios
- **stdio**: For local integrations, command-line tools
- Avoid SSE (deprecated in favor of streamable HTTP)

## Tool Naming and Design

1. Use snake_case: `search_users`, `create_project`
2. Include service prefix to avoid conflicts: `slack_send_message` not `send_message`
3. Be action-oriented: start with verbs (get, list, search, create)
4. Tool descriptions must narrowly and unambiguously describe functionality
5. Provide tool annotations (readOnlyHint, destructiveHint, idempotentHint, openWorldHint)

## Response Formats

### JSON Format
- Machine-readable structured data
- Include all available fields and metadata
- Consistent field names and types

### Markdown Format (typically default)
- Human-readable formatted text
- Use headers, lists, and formatting
- Convert timestamps to human-readable format
- Show display names with IDs in parentheses

## Pagination

- Always respect the `limit` parameter
- Implement offset or cursor-based pagination
- Return pagination metadata: `has_more`, `next_offset`/`next_cursor`, `total_count`
- Never load all results into memory
- Default to 20-50 items

## Transport Selection

| Criterion | stdio | Streamable HTTP |
|-----------|-------|-----------------|
| Deployment | Local | Remote |
| Clients | Single | Multiple |
| Complexity | Low | Medium |
| Real-time | No | Yes |

## Security Best Practices

- Use OAuth 2.1 or API keys stored in environment variables
- Sanitize file paths to prevent directory traversal
- Validate URLs and external identifiers
- Use schema validation (Pydantic/Zod) for all inputs
- Don't expose internal errors to clients
- For local HTTP servers: enable DNS rebinding protection, bind to 127.0.0.1

## Tool Annotations

| Annotation | Type | Default | Description |
|-----------|------|---------|-------------|
| `readOnlyHint` | boolean | false | Tool does not modify its environment |
| `destructiveHint` | boolean | true | Tool may perform destructive updates |
| `idempotentHint` | boolean | false | Repeated calls have no additional effect |
| `openWorldHint` | boolean | true | Tool interacts with external entities |

## Error Handling

- Use standard JSON-RPC error codes
- Report tool errors within result objects (not protocol-level errors)
- Provide helpful, specific error messages with suggested next steps
- Clean up resources properly on errors
