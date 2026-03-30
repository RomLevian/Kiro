# TypeScript MCP Server Implementation Guide

## Quick Reference

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";
```

## Server Naming
Format: `{service}-mcp-server` (e.g., `github-mcp-server`)

## Project Structure

```
{service}-mcp-server/
├── package.json
├── tsconfig.json
├── src/
│   ├── index.ts          # Main entry with McpServer init
│   ├── types.ts           # TypeScript interfaces
│   ├── tools/             # Tool implementations
│   ├── services/          # API clients and utilities
│   ├── schemas/           # Zod validation schemas
│   └── constants.ts       # Shared constants
└── dist/                  # Built JS (entry: dist/index.js)
```

## Tool Registration

Use `server.registerTool()` (NOT deprecated `server.tool()`):

```typescript
server.registerTool(
  "service_tool_name",
  {
    title: "Tool Display Name",
    description: "Comprehensive description with Args, Returns, Examples",
    inputSchema: { param: z.string().describe("Parameter description") },
    outputSchema: { result: z.string() },
    annotations: {
      readOnlyHint: true,
      destructiveHint: false,
      idempotentHint: true,
      openWorldHint: true
    }
  },
  async ({ param }) => {
    const output = { result: `Processed: ${param}` };
    return {
      content: [{ type: "text", text: JSON.stringify(output) }],
      structuredContent: output
    };
  }
);
```

## Zod Schemas

```typescript
const SearchSchema = z.object({
  query: z.string().min(2).max(200).describe("Search string"),
  limit: z.number().int().min(1).max(100).default(20).describe("Max results"),
  offset: z.number().int().min(0).default(0).describe("Pagination offset"),
  response_format: z.nativeEnum(ResponseFormat).default(ResponseFormat.MARKDOWN)
}).strict();
```

## Transport Options

```typescript
// stdio (local)
const transport = new StdioServerTransport();
await server.connect(transport);

// Streamable HTTP (remote)
import { StreamableHTTPServerTransport } from "@modelcontextprotocol/sdk/server/streamableHttp.js";
import express from "express";

const app = express();
app.post('/mcp', async (req, res) => {
  const transport = new StreamableHTTPServerTransport({
    sessionIdGenerator: undefined,
    enableJsonResponse: true
  });
  res.on('close', () => transport.close());
  await server.connect(transport);
  await transport.handleRequest(req, res, req.body);
});
```

## Error Handling

```typescript
function handleApiError(error: unknown): string {
  if (error instanceof AxiosError) {
    if (error.response) {
      switch (error.response.status) {
        case 404: return "Error: Resource not found. Check the ID.";
        case 403: return "Error: Permission denied.";
        case 429: return "Error: Rate limit exceeded. Wait and retry.";
        default: return `Error: API failed with status ${error.response.status}`;
      }
    }
  }
  return `Error: ${error instanceof Error ? error.message : String(error)}`;
}
```

## package.json

```json
{
  "name": "{service}-mcp-server",
  "version": "1.0.0",
  "type": "module",
  "main": "dist/index.js",
  "scripts": {
    "start": "node dist/index.js",
    "build": "tsc",
    "dev": "tsx watch src/index.ts"
  },
  "dependencies": {
    "@modelcontextprotocol/sdk": "^1.6.1",
    "axios": "^1.7.9",
    "zod": "^3.23.8"
  },
  "devDependencies": {
    "@types/node": "^22.10.0",
    "tsx": "^4.19.2",
    "typescript": "^5.7.2"
  }
}
```

## tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "Node16",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "declaration": true,
    "sourceMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

## Quality Checklist

- [ ] All tools use `registerTool` with title, description, inputSchema, annotations
- [ ] Zod schemas with `.strict()` enforcement and descriptive error messages
- [ ] Comprehensive descriptions with Args, Returns, Examples sections
- [ ] No `any` types — use `unknown` or proper types
- [ ] Async functions have explicit `Promise<T>` return types
- [ ] Common functionality extracted into reusable functions
- [ ] `npm run build` completes successfully
- [ ] CHARACTER_LIMIT constant for response truncation
- [ ] Pagination implemented where applicable
