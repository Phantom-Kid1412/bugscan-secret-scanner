# AI Secret Scanner API

AI Secret Scanner API 是一个基于 FastAPI 的确定性敏感信息扫描服务，用于扫描源码、日志、配置文本中的硬编码凭证、API Key、密码、私钥、PII 与高熵可疑字符串。

## Agent Discovery

- Agent Card: https://api.bugscan.cn/.well-known/agent.json
- 执行接口：https://api.bugscan.cn/api/v1/scan/check
- MCP JSON-RPC 地址：https://api.bugscan.cn/api/v1/scan/mcp
- MCP SSE 地址：https://api.bugscan.cn/api/v1/scan/mcp/sse

## MCP 联调说明

服务提供轻量级 MCP 端点，便于 Claude Desktop、Cursor、Glama.ai 以及其他原生 MCP 客户端进行发现与联调。

- 传输方式：
  - 通过 /api/v1/scan/mcp 使用 HTTP POST 承载 JSON-RPC 2.0
  - 可选的 /api/v1/scan/mcp/sse 用于端点发现和保活
- 当前支持的方法：
  - initialize
  - tools/list
  - tools/call
- 当前暴露的工具：
  - scan_secrets

MCP 端点本身不会直接执行付费扫描。它只负责暴露发现信息，并在 tools/call 中提示调用方改用 /api/v1/scan/check 这个 HTTP 接口完成真实扫描。

## 接口地址

- 请求方法：POST
- 请求路径：/api/v1/scan/check
- 完整地址：https://api.bugscan.cn/api/v1/scan/check
- 内容类型：application/json

## 请求示例

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

## 响应示例

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

## 计费说明

- 基础价格：0.01 元
- 增量价格：每 1000 有效行加收 0.01 元
- 有效行公式：max(真实行数, ceil(UTF-8 字节数 / 80))
- 免费额度：每个 IP 每天前 3 次调用免费

## 本地开发

```bash
python -m venv .venv
pip install -r requirements.txt
uvicorn app.main:app --reload
```

交互式 API 文档地址：

```text
http://127.0.0.1:8001/api/v1/scan/docs
```
