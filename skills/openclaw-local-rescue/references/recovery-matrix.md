# OpenClaw 本机救援矩阵

先执行标准探测：

```bash
openclaw gateway status
lsof -nP -iTCP:18789 -sTCP:LISTEN
openclaw models status --plain
openclaw dashboard --no-open
```

必要时追加：

```bash
openclaw models list
openclaw agent --to +10000000000 --message '请只回复OK，不要解释。' --json --timeout 60
```

## 主判断表

| 证据 | 结论 | 处理动作 |
|------|------|----------|
| `LaunchAgent (not loaded)`、`Service not installed`、`Service unit not found` | 本机 gateway 服务未安装或未加载 | 执行 `openclaw gateway install --force`，然后重跑标准探测 |
| `gateway status` 不健康，且 `18789` 没有监听 | gateway 没起来或已经掉线 | 先回读 `gateway status`，再按 LaunchAgent 路线修复；不要先动模型配置 |
| `18789` 正在监听，`RPC probe: ok`，但用户仍说 UI 没响应 | 后端大概率健康，优先怀疑旧网页会话残留 | 转到 `browser-session-cleanup.md` |
| `models status --plain` 不是 `bailian/kimi-k2.5` | 默认模型漂移 | 按“模型修复”步骤恢复默认模型 |
| `openclaw agent ...` 也能正常返回 | gateway、模型和调用链都基本正常 | 如果网页仍异常，优先按前端会话残留处理 |

## LaunchAgent 修复

当 `gateway status` 出现未加载或未安装时，标准修复命令是：

```bash
openclaw gateway install --force
```

修复后立刻回读：

```bash
openclaw gateway status
lsof -nP -iTCP:18789 -sTCP:LISTEN
openclaw dashboard --no-open
```

预期结果：

- `Service: LaunchAgent (loaded)`
- `Runtime: running`
- `RPC probe: ok`
- `127.0.0.1:18789` 正在监听

## 模型修复

先查看当前模型与 fallback：

```bash
openclaw models status --plain
openclaw models list
openclaw models fallbacks list
```

如果默认模型不是 `bailian/kimi-k2.5`，执行：

```bash
openclaw models set bailian/kimi-k2.5
openclaw models status --plain
```

如果用户希望保留常用 fallback，且 `glm-5/glm-5` 缺失，再补：

```bash
openclaw models fallbacks add glm-5/glm-5
openclaw models fallbacks list
```

不要默认清空所有 fallbacks；只有在用户明确要求重建时，才使用 `fallbacks clear`。

## 日志确认

常见日志文件通常在：

```bash
ls -1t /tmp/openclaw/openclaw-*.log | head
```

如果要快速看错误：

```bash
tail -n 200 /tmp/openclaw/openclaw-$(date +%Y-%m-%d).log
```

如果怀疑网页权限残留：

```bash
rg -n "missing scope: operator.read" /tmp/openclaw/openclaw-*.log
```

## 最终对外结论模板

- 根因：一句话写清是 LaunchAgent、模型漂移，还是旧 UI 会话残留
- 证据：列出 2 到 4 条最关键输出
- 动作：写明已执行或建议执行的命令
- 结果：写明是否已恢复监听、默认模型、dashboard URL
- 下一步：若仍异常，指向更具体的 reference 或人工确认点
