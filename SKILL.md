---
name: cpa-codex-auth-sweep-cliproxy
description: 通过 CLI Proxy API 拉取 Codex 授权数据并进行高并发探活扫描的技能。适用于「扫号」「清死号」「清理 Codex 401」「扫描凭证」等场景；支持仅扫描和明确授权后的 401 清理。
---

# 技能说明

此技能用于：

1. 通过 **CLI Proxy Management API 的认证文件接口** 获取授权（`/v0/management/auth-files`）
2. 使用 **管理端 API Call 能力**（`/v0/management/api-call` + `auth_index + $TOKEN$`）探测每个 Codex 授权状态（对齐 CLI Proxy 的刷新/代理链路）
3. 识别 401/失效凭证并在用户明确要求时清理

## 交互要求（必须）

在每次准备执行扫描前，必须先主动向用户询问并确认：

- `base_url`（CLI Proxy 管理端地址）
- `management_key`（管理密钥）

如果用户未提供这两个参数，禁止开始扫描；应先提示用户补全。

## 执行入口

```bash
python3 <SKILL目录>/scripts/cliproxy_scanner.py \
  --base-url "<CLI_PROXY_BASE_URL>" \
  --management-key "<MANAGEMENT_KEY>" \
  --output-json
```

常见用法：

```bash
# 只扫描，不删除
python3 <SKILL目录>/scripts/cliproxy_scanner.py \
  --base-url "<CLI_PROXY_BASE_URL>" \
  --management-key "<MANAGEMENT_KEY>" \
  --output-json

# 扫描 + 删除 401（需要明确删除意图）
python3 <SKILL目录>/scripts/cliproxy_scanner.py \
  --base-url "<CLI_PROXY_BASE_URL>" \
  --management-key "<MANAGEMENT_KEY>" \
  --output-json --delete-401 --yes
```

## 必要环境变量

- `CLIPROXY_BASE_URL`：CLI Proxy API 管理端地址（例：`http://localhost:8317`）
- `CLIPROXY_MANAGEMENT_KEY`：管理密钥（Management Key）

可选：

- `CLIPROXY_AUTH_FILES_ENDPOINT`：认证文件列表接口（默认：`/v0/management/auth-files`）
- `CLIPROXY_API_CALL_ENDPOINT`：管理 API Call 接口（默认：`/v0/management/api-call`）
- `CLIPROXY_AUTH_DELETE_ENDPOINT`：认证文件删除接口（默认：`/v0/management/auth-files`，通过 `?name=` 删除）
- `CODEX_PROBE_URL`：Codex 探活 URL（默认：`https://chatgpt.com/backend-api/codex/responses`）
- `SCAN_WORKERS`：并发数（默认：200）

## 判定口径（已对齐）

- 失效：HTTP 401 / invalid auth / revoked
- 额度为 0：**仅指周限额为 0**（weekly/week/周限额 等语义 + quota/limit exhausted）
- 同时输出管理端视角指标：`management_quota_exhausted`（来自 `/auth-files` 的 `unavailable + status_message(quota)`）
- 网络错误、超时、解析错误：不归类为失效或周限额 0

## 执行纪律

- 用户只说“看看/扫一下”时：只扫描，不删除。
- 只有在用户明确表达“删掉/清理/扬了”等意图时，才允许 `--delete-401`。
- 汇报优先给出汇总统计（total / ok / 401 / exceeded / error）。
