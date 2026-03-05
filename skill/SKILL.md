---
name: build-tools-for-ai-agents
description: Comprehensive guide for building effective AI Agent tools following Anthropic best practices and MCP specification standards
---

# Building Tools for AI Agents

This skill provides comprehensive guidance for building effective AI Agent tools following Anthropic's engineering best practices and the Model Context Protocol (MCP) specification standards.

## Understanding Tools

### What is a Tool

A tool is a new type of software that represents a contract between deterministic systems and non-deterministic agents. Traditional software development establishes contracts between deterministic systems (e.g., `getWeather("NYC")` always fetches weather in the same way). However, tools must account for the possibility that an agent might:

- Call the tool
- Answer from general knowledge
- Ask clarifying questions first
- Hallucinate or fail to understand how to use the tool

The core principle is to design tools for agents, not for other developers or systems.

### Tool Development Workflow

The recommended workflow for building tools follows an iterative cycle:

1. **Build a Prototype**: Quickly create a tool prototype and test it locally. Package the tool in a local MCP server or Desktop extension (DXT) with the required software libraries, APIs, or SDKs documented (preferably in `llms.txt` format).

2. **Run an Evaluation**: Generate evaluation tasks that require multiple tool calls. Use a simple agentic loop (a `while` loop wrapping alternating LLM API and tool calls). Collect metrics including top-level accuracy, total runtime, tool call count, token consumption, and tool errors.

3. **Analyze and Improve**: Stitch together evaluation agent records and paste them into Claude Code for analysis. Use the insights to batch-refactor tools and improve their effectiveness.

4. **Repeat**: Continue iterating through this cycle to refine tools.

## Core Principles for Effective Tools

### Principle 1: Choose the Right Tools

More tools do not necessarily lead to better results. Agents have unique "affordances" — LLM agents have limited context, while computer memory is cheap and abundant. The key is to design tools that accomplish specific tasks rather than exposing raw data.

**Anti-pattern (Bad)**:
```python
# Returns all contacts, forcing the agent to read through each token
list_contacts()  # Returns all contacts
```

**Best Practice (Good)**:
```python
# Directly locates the relevant contact
search_contacts(query="Jane")
message_contact(name="Jane")
```

**Tool Integration Recommendations**:

| Not Recommended | Recommended |
|-----------------|-------------|
| `list_users`, `list_events`, `create_event` | `schedule_event` (finds available time and schedules meeting) |
| `read_logs` | `search_logs` (returns only relevant log lines and context) |
| `get_customer_by_id`, `list_transactions`, `list_notes` | `get_customer_context` (integrates all customer information) |

### Principle 2: Namespace Your Tools

When agents have access to dozens of MCP servers and hundreds of tools, namespacing helps establish boundaries. Use service prefixes or resource prefixes to organize tools:

- Service prefix: `asana_search`, `jira_search`
- Resource prefix: `asana_projects_search`, `asana_users_search`

The naming convention (prefix/suffix) significantly impacts tool usage evaluation and may vary effectiveness across different LLMs. Choose based on your evaluation results.

### Principle 3: Return Meaningful Context

Return only high-signal information. Prioritize contextual relevance over flexibility. Avoid low-level technical identifiers like `uuid`, `256px_image_url`, or `mime_type`. Instead, prefer semantic fields like `name`, `image_url`, and `file_type`.

**Key Finding**: Converting arbitrary alphanumeric UUIDs into semantic language significantly improves retrieval accuracy and reduces hallucinations.

**Response Format Flexibility**: Consider adding a `response_format` parameter to allow agents to choose between detailed and concise responses:

```typescript
enum ResponseFormat {
  DETAILED = "detailed",
  CONCISE = "concise"
}

// Tool parameter example
response_format: ResponseFormat
```

- `DETAILED`: Returns complete information (approximately 206 tokens), including IDs like `thread_ts`, `channel_id`, `user_id`
- `CONCISE`: Returns only content (approximately 72 tokens), without IDs
- This approach can save approximately 2/3 of tokens

### Principle 4: Optimize for Token Efficiency

Implement these optimization strategies:

1. **Pagination**: Limit the number of items returned per call
2. **Range Selection**: Specify the range of data to return
3. **Filtering**: Return only relevant data based on query parameters
4. **Truncation**: Set reasonable default limits

Claude Code defaults to limiting tool responses to 25,000 tokens. Design your tools with this constraint in mind.

**Error Response Optimization**:

```typescript
// Bad error response
"Error: Invalid input"

// Good error response
"Invalid input. Please provide a valid email address in the format: user@example.com"
```

### Principle 5: Prompt Engineer Tool Descriptions

Write tool descriptions as if describing a new employee's job. Clearly explain implied context (professional query formats, niche terminology definitions, resource relationships). Use strict data models to explicitly describe inputs and outputs.

**Parameter Naming**: Use explicit parameter names — `user_id` is better than `user`.

**Impact**: Even small improvements to tool descriptions can yield significant performance gains. Claude Sonnet 3.5 achieved state-of-the-art performance on SWE-bench Verified precisely through precise tool description tuning.

## MCP Tool Definition Structure

### Required Fields

Every MCP tool definition must include these fields:

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Unique identifier for the tool |
| `description` | Yes | Human-readable description of functionality |
| `inputSchema` | Yes | JSON Schema defining expected parameters |

### Optional Fields

| Field | Required | Description |
|-------|----------|-------------|
| `title` | No | Human-readable name for display |
| `outputSchema` | No | JSON Schema defining expected output structure |
| `annotations` | No | Optional properties describing tool behavior |

### Tool Schema Example

```json
{
  "name": "get_weather",
  "title": "Weather Information Provider",
  "description": "Get current weather information for a specified location. This tool queries the weather API and returns temperature, conditions, and humidity data.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "location": {
        "type": "string",
        "description": "City name or zip code (e.g., 'New York' or '10001')"
      },
      "units": {
        "type": "string",
        "enum": ["celsius", "fahrenheit"],
        "description": "Temperature unit to return",
        "default": "fahrenheit"
      }
    },
    "required": ["location"]
  },
  "outputSchema": {
    "type": "object",
    "properties": {
      "temperature": {
        "type": "number",
        "description": "Current temperature in specified units"
      },
      "conditions": {
        "type": "string",
        "description": "Weather conditions description (e.g., 'Sunny', 'Cloudy', 'Rainy')"
      },
      "humidity": {
        "type": "number",
        "description": "Humidity percentage (0-100)"
      }
    },
    "required": ["temperature", "conditions", "humidity"]
  },
  "annotations": {
    "destructive": false,
    "openWorld": false
  }
}
```

## Tool Annotations

Tool annotations help disclose important characteristics about tools:

| Annotation | Description |
|------------|-------------|
| `destructive` | Indicates if the tool makes destructive changes |
| `openWorld` | Indicates if the tool requires open-world access |

## Input Schema Best Practices

### Parameter Description Requirements

Every parameter in your `inputSchema` must include a `description` field that explains:

- The format expected (e.g., "Email address in the format: user@example.com")
- Valid values or ranges (e.g., "Integer between 1 and 100")
- Whether it's optional and what the default is
- Context about what the parameter represents

**Minimum Description Length**: Tool descriptions should be comprehensive. A good description typically contains at least 50 characters and clearly explains the parameter's purpose, format, and valid values.

### Type Specifications

Always specify precise types for parameters:

```json
// Good - precise type
"user_id": {
  "type": "string",
  "description": "Unique identifier for the user (UUID format)"
}

// Good - enum for limited values
"status": {
  "type": "string",
  "enum": ["pending", "active", "completed"],
  "description": "Current status of the task"
}

// Good - array type
"tags": {
  "type": "array",
  "items": {
    "type": "string"
  },
  "description": "List of tags to filter by"
}
```

## Error Handling

### Tool Execution Errors

When a tool encounters an error, return a descriptive error message that helps the agent understand and potentially recover:

```python
try:
    result = api.call()
    return result
except ValidationError as e:
    return f"Validation error: {str(e)}. Please check that your input follows the correct format."
except RateLimitError as e:
    return f"Rate limit exceeded. Please wait {e.retry_after} seconds before retrying."
except Exception as e:
    return f"Error executing tool: {str(e)}. This may be a temporary issue. Try again or check the API status."
```

### Protocol Errors

For JSON-RPC protocol errors, use standard error codes:

```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "error": {
    "code": -32602,
    "message": "Invalid params: missing required parameter 'location'"
  }
}
```

## Tool Result Content Types

MCP supports multiple content types for tool results:

### Text Content

```json
{
  "type": "text",
  "text": "Tool result text"
}
```

### Structured Content

When returning structured data, include both text and structured content:

```json
{
  "type": "text",
  "text": "{\"temperature\": 22.5, \"conditions\": \"Partly cloudy\", \"humidity\": 65}"
},
"structuredContent": {
  "temperature": 22.5,
  "conditions": "Partly cloudy",
  "humidity": 65
}
```

### Image and Audio Content

Tools can also return image or audio content using appropriate mime types:

```json
{
  "type": "image",
  "data": "base64-encoded-data",
  "mimeType": "image/png"
}
```

## Security Considerations

### Server-side Requirements

- Validate all tool inputs
- Implement appropriate access controls
- Rate limit tool calls
- Sanitize tool outputs

### Client-side Recommendations

- Prompt user confirmation for sensitive operations
- Display tool inputs to users before calling servers
- Validate tool results before passing to LLMs
- Implement timeouts for tool calls
- Log tool usage for auditing

## Evaluation Best Practices

### Strong Task Examples (Require Multiple Tool Calls)

These tasks require agents to chain multiple tool calls and reason about the results:

- "Schedule a meeting with Jane next week to discuss our latest Acme Corp project. Attach the notes from our last project planning meeting and reserve a conference room."
- "Customer ID 9182 reported that they were charged three times for a single purchase attempt. Find all relevant log entries and determine if any other customers were affected by the same issue."

### Weak Task Examples (Too Simple)

These tasks are too simple and don't properly evaluate tool effectiveness:

- "Schedule a meeting with jane@acme.corp next week."
- "Search the payment logs for `purchase_complete` and `customer_id=9182`."

### Metrics to Collect

- Top-level accuracy
- Total runtime
- Tool call count
- Token consumption
- Tool errors

## Summary: Effective Tool Characteristics

The core characteristics of effective AI agent tools are:

1. **Clear Intent**: Tools should have well-defined purposes and explicit boundaries
2. **Context Efficiency**: Tools should be designed to minimize unnecessary context while providing all relevant information
3. **Composability**: Tools should be designed to work together in diverse workflows
4. **Actionability**: Tools should enable agents to intuitively solve real-world tasks

Follow this development process: Build Prototype → Run Evaluation → Analyze Results → Improve Tools → Repeat

## Reference Resources

- [Model Context Protocol Documentation](https://modelcontextprotocol.io/docs/getting-started/intro)
- [MCP SDK](https://modelcontextprotocol.io/docs/sdk)
- [Anthropic API Documentation](https://docs.anthropic.com/llms.txt)
- [Tool Evaluation Cookbook](https://github.com/anthropics/anthropic-cookbook/blob/main/tool_evaluation/tool_evaluation.ipynb)
- [Developer Guide - Tool Definitions](https://docs.anthropic.com/en/docs/agents-and-tools/tool-use/implement-tool-use)
