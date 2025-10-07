# Server Configuration

## Quick Start

1. Copy the example configuration file:
   ```bash
   cp start-server.sh.example start-server.sh
   ```

2. Edit `start-server.sh` with your actual credentials:
   ```bash
   nano start-server.sh  # or use your preferred editor
   ```

3. Update the following values:
   - `DATABASE_URL`: Your PostgreSQL or MySQL connection string
   - `OPENAI_API_KEY`: Your OpenAI API key (optional, required for AI features)
   - `OPENAI_BASE_URL`: Your OpenAI proxy URL (if using a proxy)
   - `OPENAI_USER`: Your username (required by some proxies like NetApp)

4. Run the server:
   ```bash
   ./start-server.sh > server.log 2>&1 &
   ```

## Configuration Options

### Required

- **DATABASE_URL**: Database connection string
  - PostgreSQL: `postgres://user:pass@host:5432/dbname`
  - MySQL: `mysql://user:pass@tcp(host:3306)/dbname`

### Optional

- **OPENAI_API_KEY**: API key for OpenAI or compatible service
- **OPENAI_MODEL**: Model to use (default: `gpt-4o-mini`)
- **OPENAI_BASE_URL**: Base URL for OpenAI API (default: `https://api.openai.com`)
- **OPENAI_USER**: User identifier (required by some proxies)
- **HTTP_ADDR**: Server listen address (default: `:8080`)
- **HTTP_PATH**: MCP endpoint path (default: `/mcp`)
- **AUTH_BEARER**: Bearer token for authentication
- **SCHEMA_TTL**: Schema cache TTL (default: `5m`)
- **QUERY_TIMEOUT**: Query execution timeout (default: `25s`)
- **MAX_ROWS**: Maximum rows per query (default: `200`)
- **LOG_LEVEL**: Logging level: `debug`, `info`, `warn`, `error` (default: `info`)

## Security Note

⚠️ **Important**: The `start-server.sh` file is gitignored to prevent accidentally committing sensitive credentials. Always use `start-server.sh.example` as a template and never commit the actual `start-server.sh` file with real credentials.

## Alternative: Environment Variables

Instead of using the startup script, you can export environment variables directly:

```bash
export DATABASE_URL="postgres://user:pass@localhost:5432/mydb"
export OPENAI_API_KEY="your-key"
export OPENAI_BASE_URL="https://api.openai.com"
export OPENAI_USER="your-username"

./pgmcp-server
```

## Troubleshooting

### Port already in use
```bash
# Kill existing server
pkill -f pgmcp-server

# Or use a different port
export HTTP_ADDR=":8081"
./start-server.sh
```

### Database connection issues
```bash
# Test your database connection
psql "$DATABASE_URL"  # For PostgreSQL
mysql -u username -p database_name  # For MySQL
```

### OpenAI API errors
- Verify your API key is correct
- Check if your proxy requires a `user` field (set `OPENAI_USER`)
- Ensure `OPENAI_BASE_URL` points to the correct endpoint
