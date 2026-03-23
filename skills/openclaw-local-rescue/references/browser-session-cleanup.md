# 旧网页会话残留

当后端已经健康，但用户仍说 Control UI “没有响应”，优先考虑旧浏览器会话或残缺 scope 残留。

## 常见特征

- `openclaw gateway status` 显示健康
- `lsof -nP -iTCP:18789 -sTCP:LISTEN` 显示 `127.0.0.1:18789` 正在监听
- `openclaw models status --plain` 正常，默认模型仍是 `bailian/kimi-k2.5`
- `openclaw agent --json` 可以正常返回
- 日志里出现 `missing scope: operator.read`

快速检索日志：

```bash
rg -n "missing scope: operator.read" /tmp/openclaw/openclaw-*.log
```

## 首选处理动作

先给用户一个新的 dashboard 地址：

```bash
openclaw dashboard --no-open
```

然后让用户按这个顺序操作：

1. 关闭所有旧的 `127.0.0.1:18789` 标签页
2. 用新输出的 dashboard URL 重新打开
3. 如果新页面可用，就把问题归因到旧网页会话残留

## 进一步动作

如果新地址仍异常，再做这两类检查：

```bash
tail -n 200 /tmp/openclaw/openclaw-$(date +%Y-%m-%d).log
test -f ~/.openclaw/paired.json && sed -n '1,220p' ~/.openclaw/paired.json
```

此时可以告诉用户：

- stale device 可能缺少 `operator.read` 或 `operator.write`
- 这通常是旧 UI 会话问题，不代表 gateway 又挂了

## 安全边界

不要默认删除 `~/.openclaw/paired.json` 或其中的设备记录。

只有在以下条件同时满足时，才建议进一步清理：

- 后端健康且可稳定调用
- 新 dashboard URL 仍持续异常
- 日志明确指向 scope 缺失
- 用户同意做会话层清理

在未获得确认前，默认停留在“关闭旧标签页并重新打开新地址”这一层。
