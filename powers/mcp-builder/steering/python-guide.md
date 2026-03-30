# Python MCP Server Implementation Guide

## Quick Reference

```python
from mcp.server.fastmcp import FastMCP
from pydantic import BaseModel, Field, field_validator, ConfigDict
from typing import Optional, List, Dict, Any
from enum import Enum
import httpx
```

## Server Naming
Format: `{service}_mcp` (e.g., `github_mcp`)

## Server Initialization

```python
mcp = FastMCP("service_mcp")
```

## Tool Registration

```python
@mcp.tool(
    name="service_tool_name",
    annotations={
        "title": "Human-Readable Title",
        "readOnlyHint": True,
        "destructiveHint": False,
        "idempotentHint": True,
        "openWorldHint": True
    }
)
async def service_tool_name(params: InputModel) -> str:
    '''Tool description becomes the description field.

    Args:
        params (InputModel): Validated input containing:
            - query (str): Search string
            - limit (Optional[int]): Max results (default: 20)

    Returns:
        str: JSON-formatted response with schema:
        {
            "total": int,
            "items": [...]
        }

    Examples:
        - "Find users" -> params with query="john"
    '''
    pass
```

## Pydantic v2 Models

```python
class UserSearchInput(BaseModel):
    model_config = ConfigDict(
        str_strip_whitespace=True,
        validate_assignment=True,
        extra='forbid'
    )

    query: str = Field(..., description="Search string", min_length=2, max_length=200)
    limit: Optional[int] = Field(default=20, description="Max results", ge=1, le=100)
    offset: Optional[int] = Field(default=0, description="Pagination offset", ge=0)
    response_format: ResponseFormat = Field(
        default=ResponseFormat.MARKDOWN,
        description="Output format: 'markdown' or 'json'"
    )

    @field_validator('query')
    @classmethod
    def validate_query(cls, v: str) -> str:
        if not v.strip():
            raise ValueError("Query cannot be empty")
        return v.strip()
```

Key Pydantic v2 rules:
- Use `model_config` not nested `Config` class
- Use `field_validator` not deprecated `validator`
- Use `model_dump()` not deprecated `dict()`
- Validators require `@classmethod` decorator

## Response Formats

```python
class ResponseFormat(str, Enum):
    MARKDOWN = "markdown"
    JSON = "json"
```

## Error Handling

```python
def _handle_api_error(e: Exception) -> str:
    if isinstance(e, httpx.HTTPStatusError):
        status = e.response.status_code
        if status == 404: return "Error: Resource not found. Check the ID."
        if status == 403: return "Error: Permission denied."
        if status == 429: return "Error: Rate limit exceeded. Wait and retry."
        return f"Error: API failed with status {status}"
    if isinstance(e, httpx.TimeoutException):
        return "Error: Request timed out. Try again."
    return f"Error: {type(e).__name__}"
```

## Shared API Client

```python
async def _make_api_request(endpoint: str, method: str = "GET", **kwargs) -> dict:
    async with httpx.AsyncClient() as client:
        response = await client.request(
            method,
            f"{API_BASE_URL}/{endpoint}",
            timeout=30.0,
            **kwargs
        )
        response.raise_for_status()
        return response.json()
```

## Advanced Features

### Context Injection

```python
from mcp.server.fastmcp import FastMCP, Context

@mcp.tool()
async def advanced_tool(query: str, ctx: Context) -> str:
    await ctx.report_progress(0.5, "Processing...")
    await ctx.log_info("Query received", {"query": query})
    return result
```

### Resources

```python
@mcp.resource("file://documents/{name}")
async def get_document(name: str) -> str:
    with open(f"./docs/{name}", "r") as f:
        return f.read()
```

### Lifespan Management

```python
from contextlib import asynccontextmanager

@asynccontextmanager
async def app_lifespan():
    db = await connect_to_database()
    yield {"db": db}
    await db.close()

mcp = FastMCP("service_mcp", lifespan=app_lifespan)
```

### Transport

```python
# stdio (default, local)
if __name__ == "__main__":
    mcp.run()

# Streamable HTTP (remote)
if __name__ == "__main__":
    mcp.run(transport="streamable_http", port=8000)
```

## Quality Checklist

- [ ] All tools have `name` and `annotations` in decorator
- [ ] Pydantic BaseModel with Field() definitions for all inputs
- [ ] Comprehensive docstrings with Args, Returns, Examples
- [ ] All async functions use `async def`
- [ ] httpx with async context managers for HTTP calls
- [ ] Type hints throughout
- [ ] Common functionality extracted into reusable functions
- [ ] Server name follows `{service}_mcp` format
- [ ] Constants at module level in UPPER_CASE
