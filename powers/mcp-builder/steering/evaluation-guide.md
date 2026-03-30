# MCP Server Evaluation Guide

## Overview

Create 10 comprehensive evaluation questions to test whether LLMs can effectively use your MCP server to answer realistic, complex questions using only the tools provided.

## Requirements

- 10 human-readable questions
- Questions must be READ-ONLY, INDEPENDENT, NON-DESTRUCTIVE
- Each question requires multiple tool calls (potentially dozens)
- Answers must be single, verifiable values
- Answers must be STABLE (won't change over time)

## Question Guidelines

1. Questions MUST be independent — no dependencies between them
2. Questions MUST require only non-destructive, idempotent operations
3. Questions must be realistic, clear, concise, and complex
4. Require deep exploration with multi-hop reasoning
5. May require extensive paging through results
6. Must not be solvable with straightforward keyword search
7. Should stress-test tool return values across data types (IDs, timestamps, URLs, etc.)
8. Should reflect real human use cases
9. May include ambiguous questions with a single verifiable answer
10. Answers must NOT change over time — use historical/closed data

## Answer Guidelines

- Verifiable via direct string comparison
- Specify output format in the question (e.g., "Use YYYY/MM/DD", "Answer True or False")
- Prefer human-readable formats (names, dates, URLs) over opaque IDs
- Must be stable/stationary — based on closed concepts
- Must be diverse across modalities (user IDs, timestamps, file names, etc.)
- Must NOT be complex structures or lists

## Output Format

```xml
<evaluation>
   <qa_pair>
      <question>Your question here</question>
      <answer>Single verifiable answer</answer>
   </qa_pair>
</evaluation>
```

## Evaluation Process

1. Study API documentation and available tools
2. Inspect tools without calling them
3. Use READ-ONLY operations to explore content
4. Generate 10 questions following all guidelines
5. Verify answers by solving questions yourself

## Running Evaluations

```bash
pip install anthropic mcp
export ANTHROPIC_API_KEY=your_key

# stdio transport
python scripts/evaluation.py \
  -t stdio \
  -c python \
  -a my_server.py \
  evaluation.xml

# HTTP transport
python scripts/evaluation.py \
  -t http \
  -u https://example.com/mcp \
  evaluation.xml
```

## Good Question Examples

Multi-hop: "Find the repository archived in Q3 2023 that was previously the most forked. What was its primary language?"
→ Answer: `Python`

Aggregation: "Among critical bugs in January 2024, which assignee resolved the highest percentage within 48 hours? Provide username."
→ Answer: `alex_eng`

## Poor Question Examples

- "How many open issues are assigned?" (answer changes over time)
- "Find PR titled 'Add auth feature'" (too easy, keyword search)
- "List all Python repositories" (answer is a list, not verifiable by string comparison)
