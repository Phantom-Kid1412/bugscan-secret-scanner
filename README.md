# AI Secret Scanner API

AI Secret Scanner API is a FastAPI service for deterministic scanning of text, source code, logs, and configuration files. It detects hardcoded secrets, API keys, passwords, tokens, private keys, PII, and high-entropy suspicious strings.

## Agent Discovery

- Agent Card: https://api.bugscan.cn/.well-known/agent.json
- Execution endpoint: https://api.bugscan.cn/api/v1/scan/check
- MCP JSON-RPC Endpoint: https://api.bugscan.cn/api/v1/scan/mcp
- MCP SSE Endpoint: https://api.bugscan.cn/api/v1/scan/mcp/sse

## MCP Integration

The service exposes a lightweight MCP endpoint for discovery and client compatibility with Claude Desktop, Cursor, Glama.ai, and other native MCP tooling.

- Transport:
  - JSON-RPC 2.0 over HTTP POST to /api/v1/scan/mcp
  - Optional SSE stream at /api/v1/scan/mcp/sse for endpoint discovery and keep-alive
- Supported methods:
  - initialize
  - tools/list
  - tools/call
- Available tool:
  - scan_secrets

The MCP endpoint does not execute paid scans directly. It only exposes discovery metadata and returns guidance that points callers to the HTTP API at /api/v1/scan/check.

## API Endpoint

- Method: POST
- Path: /api/v1/scan/check
- Full URL: https://api.bugscan.cn/api/v1/scan/check
- Content-Type: application/json

## Request Example

```bash
curl -X POST "https://api.bugscan.cn/api/v1/scan/check" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "aws_key = \"AKIAIOSFODNN7EXAMPLE\"\nphone = \"13812345678\"",
    "scan_type": "all",
    "filename": "config.py",
    "max_findings": 20
  }'
```

## Response Example

```json
{
  "request_id": "7e0ff4c8-7987-4496-b391-b8b9f4d1b10e",
  "summary": "检测到2处硬编码密钥",
  "total_findings": 2,
  "scanned_lines": 2,
  "findings": [
    {
      "line": 1,
      "column": 12,
      "severity": "critical",
      "category": "secret",
      "message": "检测到AWS Access Key",
      "matched_text": "AKIA************MPLE",
      "suggestion": "立即移除硬编码敏感信息，改为环境变量、密钥管理服务或配置中心。"
    }
  ],
  "scan_metadata": {
    "duration_ms": 6,
    "rules_loaded": 10,
    "rules_executed": 11,
    "scan_type": "all",
    "truncated": false,
    "utf8_bytes": 52,
    "filename": "config.py"
  }
}
```

## Pricing

- Base price: 0.01 CNY
- Usage price: 0.01 CNY per 1000 effective lines
- Effective lines formula: max(real line count, ceil(UTF-8 bytes / 80))
- Free tier: first 3 calls per day per IP are free

## Local Development

```bash
python -m venv .venv
pip install -r requirements.txt
uvicorn app.main:app --reload
```

Open the interactive API docs at:

```text
http://127.0.0.1:8001/api/v1/scan/docs
```
