# Build Tools for AI Agents

A comprehensive guide for building effective AI Agent tools following Anthropic's engineering best practices and the Model Context Protocol (MCP) specification standards.

## Overview

This project provides comprehensive guidance for creating tools that AI agents can effectively use. It covers the entire tool development lifecycle from design principles to implementation best practices.

## Core Principles

### 1. Choose the Right Tools
Design tools that accomplish specific tasks rather than exposing raw data. Tools should help agents complete meaningful work, not just retrieve information.

### 2. Namespace Your Tools
Use service prefixes (e.g., `asana_search`, `jira_search`) to organize tools when agents have access to multiple MCP servers.

### 3. Return Meaningful Context
Prioritize contextual relevance over flexibility. Use semantic fields like `name`, `image_url`, and `file_type` instead of low-level identifiers like `uuid`.

### 4. Optimize for Token Efficiency
- Implement pagination
- Use range selection
- Apply filtering
- Set reasonable default limits

### 5. Prompt Engineer Tool Descriptions
Write descriptions as if describing a new employee's job. Clearly explain implied context, professional formats, and terminology.

## Development Workflow

```
Build Prototype → Run Evaluation → Analyze Results → Improve Tools → Repeat
```

1. **Build a Prototype**: Create a tool prototype and test locally. Package in a local MCP server.
2. **Run an Evaluation**: Generate evaluation tasks requiring multiple tool calls. Collect metrics.
3. **Analyze and Improve**: Review evaluation results and refine tools.
4. **Repeat**: Continue iterating to improve effectiveness.

## MCP Tool Definition

### Required Fields

| Field | Description |
|-------|-------------|
| `name` | Unique identifier for the tool |
| `description` | Human-readable description of functionality |
| `inputSchema` | JSON Schema defining expected parameters |

### Example

```json
{
  "name": "get_weather",
  "description": "Get current weather information for a specified location.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "location": {
        "type": "string",
        "description": "City name or zip code (e.g., 'New York' or '10001')"
      }
    },
    "required": ["location"]
  }
}
```

## Best Practices

### Input Schema
- Include comprehensive descriptions for all parameters (at least 50 characters)
- Specify precise types (string, number, enum, array)
- Define valid values and ranges

### Error Handling
- Return descriptive error messages that help agents understand and recover
- Use standard JSON-RPC error codes for protocol errors

### Security
- Validate all tool inputs
- Implement access controls
- Rate limit tool calls
- Sanitize outputs

## Evaluation

### Strong Task Examples
- "Schedule a meeting with Jane next week to discuss the project..."
- "Find all relevant log entries for customer ID 9182..."

### Metrics to Collect
- Top-level accuracy
- Total runtime
- Tool call count
- Token consumption
- Tool errors

## Reference Resources

- [Model Context Protocol Documentation](https://modelcontextprotocol.io/docs/getting-started/intro)
- [MCP SDK](https://modelcontextprotocol.io/docs/sdk)
- [Anthropic API Documentation](https://docs.anthropic.com/llms.txt)
- [Tool Evaluation Cookbook](https://github.com/anthropics/anthropic-cookbook/blob/main/tool_evaluation/tool_evaluation.ipynb)

## License

This guide is provided for educational purposes.
