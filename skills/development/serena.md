# Serena Web Scraping Agent

## Overview
Serena is a Neoma web scraping and content fetching agent that delegates web operations to reduce token usage in Claude Code conversations.

## Available Tools

### 1. serena_fetch_url
Fetch content from a URL using Serena's web scraping capabilities.

**Parameters:**
- `url` (string, required): The URL to fetch
- `format` (string, optional): Response format - "text", "markdown", or "json" (default: "text")

**Example:**
```
Use serena_fetch_url to get the content from https://example.com
```

### 2. serena_search
Search the web and return results.

**Parameters:**
- `query` (string, required): Search query
- `num_results` (number, optional): Number of results to return (default: 5)

**Example:**
```
Use serena_search to find information about "Claude Code MCP integration"
```

## When to Use Serena

Use Serena when you need to:
- Fetch web page content
- Scrape website data
- Search the web for information
- Retrieve documentation from URLs
- Extract content from online sources

## Token Optimization

By delegating web fetching to Serena, you reduce the amount of content that needs to be processed in the main Claude Code context, optimizing token usage.

## Technical Details

**Integration Type:** HTTP-to-MCP Bridge
**HTTP Endpoint:** http://127.0.0.1:8121
**Queue API:** POST /act (submit), GET /act/{id} (status)
**MCP Server:** Connected via stdio transport

**Current Status:**
- ✅ MCP bridge: Connected and functional
- ✅ HTTP queue API: Running on port 8121
- ⏳ Worker process: Not running (tasks queue but need worker to process)

## Architecture

```
Claude Code
    ↓ (MCP stdio)
serena-mcp-http-bridge
    ↓ (HTTP POST/GET)
serena-http-bridge.py (port 8121)
    ↓ (file queue)
~/.local/share/serena/queue/
    ↓ (worker needed)
[Web Scraping Execution]
```

## Usage Notes

1. Serena tools are available in all conversations via MCP integration
2. Tasks are queued to `~/.local/share/serena/queue/`
3. A worker process is needed to actually execute the web scraping tasks
4. Results are returned via polling the queue status endpoint

## Neoma Integration

**Agent Port:** 8121
**Agent Type:** web-scraping
**Documentation:** /media/granny/larger SSD/Neoma_project/docs/SERENA_MCP_SETUP.md
**Memory Reference:** system_config/serena_mcp_http_bridge

## Files

- Bridge: `/home/granny/.local/bin/serena-mcp-http-bridge`
- HTTP Server: `/home/granny/.local/bin/serena-http-bridge.py`
- Queue Directory: `~/.local/share/serena/queue/`
