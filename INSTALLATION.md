# Installation Guide

Detailed setup instructions for each supported MCP client, plus Docker and Kubernetes deployment.

## Claude Code

**Using uvx (recommended)**

Configure your MCP client in `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "datawrapper": {
      "command": "uvx",
      "args": ["datawrapper-mcp"],
      "env": {
        "DATAWRAPPER_ACCESS_TOKEN": "your-token-here"
      }
    }
  }
}
```

**Using pip**

First install the package:

```bash
pip install datawrapper-mcp
```

Then configure your MCP client in `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "datawrapper": {
      "command": "datawrapper-mcp",
      "env": {
        "DATAWRAPPER_ACCESS_TOKEN": "your-token-here"
      }
    }
  }
}
```

## OpenAI Codex

**CLI with uvx**

Add this to `~/.codex/config.toml`:

```toml
[mcp_servers.datawrapper]
args = ["datawrapper-mcp"]
command = "uvx"
startup_timeout_sec = 30

[mcp_servers.datawrapper.env]
DATAWRAPPER_ACCESS_TOKEN = "your-token-here"
```

**CLI with pip**

First install the package:

```bash
pip install datawrapper-mcp
```

Then add this to `~/.codex/config.toml`:

```toml
[mcp_servers.datawrapper]
command = "datawrapper-mcp"
startup_timeout_sec = 30

[mcp_servers.datawrapper.env]
DATAWRAPPER_ACCESS_TOKEN = "your-token-here"
```

**Secure secrets**

For enhanced security, you can configure a pass-through environment variable by ensuring that `DATAWRAPPER_ACCESS_TOKEN` is set in your environment, and replacing this in your `config.toml`:

```toml
[mcp_servers.datawrapper.env]
DATAWRAPPER_ACCESS_TOKEN = "your-token-here"
```

With this:

```toml
env_vars = ["DATAWRAPPER_ACCESS_TOKEN"]
```

This ensures that the value set for `DATAWRAPPER_ACCESS_TOKEN` in your environment is passed through to Codex without having to store the secret as text in a config file.

**Desktop application**

If you're using the [Codex Desktop Application](https://openai.com/codex/), you can set up the MCP in your settings under `MCP servers`:

1. Under Custom servers, click `Add server`
2. Under Name, enter `datawrapper-mcp`
3. Select STDIO
4. Under Command to launch, type `uvx` ([you must have uv installed](https://docs.astral.sh/uv/getting-started/installation/))
5. Under Arguments, add `datawrapper-mcp`
6. Under Environment variables, add `DATAWRAPPER_ACCESS_TOKEN` as the key and your token as the value
7. Click Save

## Kubernetes Deployment

For enterprise deployments, this server can be deployed to Kubernetes using HTTP transport.

### Building the Docker Image

```bash
docker build -t datawrapper-mcp:latest .
```

### Running with Docker

```bash
docker run -p 8501:8501 \
  -e DATAWRAPPER_ACCESS_TOKEN=your-token-here \
  -e MCP_SERVER_HOST=0.0.0.0 \
  -e MCP_SERVER_PORT=8501 \
  datawrapper-mcp:latest
```

### Environment Variables

- `DATAWRAPPER_ACCESS_TOKEN`: Your Datawrapper API token (required)
- `MCP_SERVER_HOST`: Server host (default: `0.0.0.0`)
- `MCP_SERVER_PORT`: Server port (default: `8501`)
- `MCP_SERVER_NAME`: Server name (default: `datawrapper-mcp`)

### Health Check Endpoint

The HTTP server includes a `/healthz` endpoint for Kubernetes liveness and readiness probes:

```bash
curl http://localhost:8501/healthz
# Returns: {"status": "healthy", "service": "datawrapper-mcp"}
```

### Discovery Endpoint

The HTTP server serves a `/.well-known/mcp.json` endpoint for MCP client auto-discovery:

```bash
curl http://localhost:8501/.well-known/mcp.json
```

### Kubernetes Configuration Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: datawrapper-mcp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: datawrapper-mcp
  template:
    metadata:
      labels:
        app: datawrapper-mcp
    spec:
      containers:
        - name: datawrapper-mcp
          image: datawrapper-mcp:latest
          ports:
            - containerPort: 8501
          env:
            - name: DATAWRAPPER_ACCESS_TOKEN
              valueFrom:
                secretKeyRef:
                  name: datawrapper-secrets
                  key: access-token
          livenessProbe:
            httpGet:
              path: /healthz
              port: 8501
            initialDelaySeconds: 5
            periodSeconds: 30
          readinessProbe:
            httpGet:
              path: /healthz
              port: 8501
            initialDelaySeconds: 5
            periodSeconds: 10
---
apiVersion: v1
kind: Service
metadata:
  name: datawrapper-mcp
spec:
  selector:
    app: datawrapper-mcp
  ports:
    - protocol: TCP
      port: 8501
      targetPort: 8501
```
