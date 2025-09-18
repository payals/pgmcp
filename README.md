[![ci](https://github.com/subnetmarco/pgmcp/actions/workflows/ci.yml/badge.svg)](https://github.com/subnetmarco/pgmcp/actions/workflows/ci.yml)
[![Go Report Card](https://goreportcard.com/badge/github.com/subnetmarco/pgmcp)](https://goreportcard.com/report/github.com/subnetmarco/pgmcp)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

# PGMCP - PostgreSQL Model Context Protocol Server

PGMCP connects AI assistants to **any PostgreSQL database** through natural language queries. Ask questions in plain English and get structured SQL results with automatic streaming and robust error handling.

**Works with**: Cursor, Claude Desktop, VS Code extensions, and any [MCP-compatible client](https://modelcontextprotocol.io/)

## Quick Start

### Prerequisites
- Go 1.23+
- PostgreSQL database
- OpenAI API key (optional, can use other compatible APIs)

### Installation

```bash
# Build
go build -o pgmcp-server ./server
go build -o pgmcp-client ./client

# Set up database (optional - works with any existing PostgreSQL database)
export DATABASE_URL="postgres://user:password@localhost:5432/mydb"
psql $DATABASE_URL < schema.sql

# Run server
export OPENAI_API_KEY="your-api-key"
./pgmcp-server

# Test with client
./pgmcp-client -ask "What tables do I have?" -format table
./pgmcp-client -ask "Show me recent records" -format table
```

### Environment Variables

**Required:**
- `DATABASE_URL`: PostgreSQL connection string

**Optional:**
- `OPENAI_API_KEY`: OpenAI API key for SQL generation
- `OPENAI_MODEL`: Model to use (default: "gpt-4o-mini")
- `HTTP_ADDR`: Server address (default: ":8080")
- `AUTH_BEARER`: Bearer token for authentication

```
👤 User / AI Assistant
         │
         │ "Who are the top customers?"
         ▼
┌─────────────────────────────────────────────────────────────┐
│                    Any MCP Client                           │
│                                                             │
│  PGMCP CLI  │  Cursor  │  Claude Desktop  │  VS Code  │ ... │
│  JSON/CSV   │  Chat    │  AI Assistant    │  Editor   │     │
└─────────────────────────────────────────────────────────────┘
         │
         │ HTTP SSE / MCP Protocol
         ▼
┌─────────────────────────────────────────────────────────────┐
│                    PGMCP Server                             │
│                                                             │
│  🔒 Security    🧠 AI Engine      🌊 Streaming               │
│  • Input Valid  • Schema Cache    • Auto-Pagination         │
│  • Audit Log    • OpenAI API      • Memory Management       │
│  • SQL Guard    • Error Recovery  • Connection Pool         │
└─────────────────────────────────────────────────────────────┘
         │
         │ Read-Only SQL Queries
         ▼
┌─────────────────────────────────────────────────────────────┐
│                Your PostgreSQL Database                     │
│                                                             │
│  Any Schema: E-commerce, Analytics, CRM, etc.               │
│  Tables • Views • Indexes • Functions                       │
└─────────────────────────────────────────────────────────────┘

External AI Services:
OpenAI API • Anthropic • Local LLMs (Ollama, etc.)

Key Benefits:
✅ Works with ANY PostgreSQL database (no assumptions about schema)
✅ No schema modifications required  
✅ Read-only access (100% safe)
✅ Automatic streaming for large results
✅ Intelligent query understanding (singular vs plural)
✅ Robust error handling (graceful AI failure recovery)
✅ PostgreSQL case sensitivity support (mixed-case tables)
✅ Production-ready security and performance
✅ Universal database compatibility
✅ Multiple output formats (table, JSON, CSV)
✅ Free-text search across all columns
✅ Authentication support
✅ Comprehensive testing suite
```

## Usage Examples

```bash
# Ask questions in natural language
./pgmcp-client -ask "What are the top 5 customers?" -format table
./pgmcp-client -ask "How many orders were placed today?" -format json

# Search across all text fields
./pgmcp-client -search "john" -format table

# Multiple questions at once
./pgmcp-client -ask "Show tables" -ask "Count users" -format table

# Different output formats
./pgmcp-client -ask "Export all data" -format csv -max-rows 1000
```

## Features

- **Natural Language to SQL**: Ask questions in plain English
- **Automatic Streaming**: Handles large result sets automatically  
- **Safe Read-Only Access**: Prevents any write operations
- **Text Search**: Search across all text columns
- **Multiple Output Formats**: Table, JSON, and CSV
- **PostgreSQL Case Sensitivity**: Handles mixed-case table names correctly
- **Universal Compatibility**: Works with any PostgreSQL database

## Example Database

The project includes an Amazon-like marketplace schema (`schema.sql`) with realistic data:
- **Users** (5,000), **"Categories"** (23), **Items** (1,800), **Orders** (10,000)
- **Mixed-case table names** for testing case sensitivity
- **Full-text search indexes** and realistic relationships

Use your own database:
```bash
export DATABASE_URL="postgres://user:pass@host:5432/your_db"
./pgmcp-server
./pgmcp-client -ask "What tables do I have?"
```

## AI Error Handling

When AI generates incorrect SQL, PGMCP handles it gracefully:

```json
{
  "error": "Column not found in generated query",
  "suggestion": "Try rephrasing your question or ask about specific tables",
  "original_sql": "SELECT non_existent_column FROM table..."
}
```

Instead of crashing, the system provides helpful feedback and continues operating.

## MCP Integration

### Cursor Integration

```bash
# Start server
export DATABASE_URL="postgres://user:pass@localhost:5432/your_db"
./pgmcp-server
```

Add to Cursor settings:
```json
{
  "mcp.servers": {
    "pgmcp": {
      "transport": {
        "type": "sse",
        "url": "http://localhost:8080/mcp/sse"
      }
    }
  }
}
```

### Claude Desktop Integration

Edit `~/.config/claude-desktop/claude_desktop_config.json`:
```json
{
  "mcpServers": {
    "pgmcp": {
      "command": "curl",
      "args": ["-N", "-H", "Accept: text/event-stream", "http://localhost:8080/mcp/sse"]
    }
  }
}
```

## PostgreSQL Case Sensitivity

PGMCP automatically handles PostgreSQL's case-sensitive identifiers:

```sql
-- Database has mixed-case table
CREATE TABLE "Categories" (id SERIAL, name TEXT);

-- PGMCP schema shows (with quotes)
TABLE public."Categories"(id integer, name text)

-- AI generates correct SQL
SELECT name FROM "Categories" WHERE id = 1;
```

**Benefits:**
- Works with any PostgreSQL naming convention
- No schema changes required
- Eliminates "relation does not exist" errors for mixed-case tables

## API Tools

- **`ask`**: Natural language questions → SQL queries with automatic streaming
- **`search`**: Free-text search across all database text columns  
- **`stream`**: Advanced streaming for very large result sets with pagination

## Safety Features

- **Read-Only Enforcement**: Blocks write operations (INSERT, UPDATE, DELETE, etc.)
- **Query Timeouts**: Prevents long-running queries
- **Input Validation**: Sanitizes and validates all user input
- **Transaction Isolation**: All queries run in read-only transactions

## Testing

```bash
# Unit tests
go test ./server -v

# Integration tests (requires PostgreSQL)
go test ./server -tags=integration -v
```

## License

Apache 2.0 - See LICENSE file for details.

## Related Projects

- [Model Context Protocol](https://modelcontextprotocol.io/) - The underlying protocol specification
- [MCP Go SDK](https://github.com/modelcontextprotocol/go-sdk) - Go implementation of MCP

---

PGMCP makes your PostgreSQL database accessible to AI assistants through natural language while maintaining security through read-only access controls.
