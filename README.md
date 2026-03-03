# cpa-codex-auth-sweep-cliproxy

基于 CLI Proxy Management API 的 Codex 授权巡检/清理技能。

## 功能

- 从管理端读取认证文件（`/v0/management/auth-files`）
- 通过管理端 `api-call` + `auth_index` + `$TOKEN$` 探测每个授权
- 统计：
  - 总数
  - 失效（401）
  - 周限额为 0（weekly quota）
  - 状态码分布
- 可选：自动删除 401 对应认证文件

## 目录结构

```text
cpa-codex-auth-sweep-cliproxy/
├── SKILL.md
├── README.md
└── scripts/
    └── cliproxy_scanner.py
```

## 使用方式

### 1) 仅扫描

```bash
python3 scripts/cliproxy_scanner.py \
  --base-url "https://your-cliproxy.example.com" \
  --management-key "YOUR_MANAGEMENT_KEY" \
  --workers 120 \
  --output-json
```

### 2) 扫描并删除 401

```bash
python3 scripts/cliproxy_scanner.py \
  --base-url "https://your-cliproxy.example.com" \
  --management-key "YOUR_MANAGEMENT_KEY" \
  --workers 120 \
  --delete-401 --yes \
  --output-json
```

### 3) 自签证书环境

```bash
python3 scripts/cliproxy_scanner.py ... --insecure
```

## 关键参数

- `--base-url`：CLI Proxy 管理端地址（必填）
- `--management-key`：管理密钥（必填）
- `--workers`：并发数（默认 80）
- `--delete-401 --yes`：启用并确认删除 401
- `--insecure`：关闭 TLS 校验（仅内网调试建议）

## 输出说明

`summary` 主要字段：

- `total`：参与探测的总数
- `unauthorized_401`：判定失效数量
- `weekly_quota_zero`：周限额为 0 数量
- `ok`：2xx 数量
- `errors`：请求失败数量
- `management_quota_exhausted`：管理端状态视角 quota exhausted 数量
- `status_code_buckets`：状态码分布
