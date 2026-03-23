---
name: "openclaw-local-rescue"
description: "从 tx-2 或其他辅助环境快速诊断和救援本机 OpenClaw。当用户说“救援mac”、本机 OpenClaw 没响应、Control UI 卡死、Kimi 2.5 默认模型疑似丢失、或需要远程协助排障时调用此 skill。"
metadata:
  short-description: "救援本机 OpenClaw，优先恢复网关与 Kimi 2.5"
---

# OpenClaw 本机救援

这个 skill 用于从 tx-2 或其他辅助环境，快速救援用户本机的 OpenClaw。

默认目标：

- 恢复本机 gateway 和 Control UI 可用性
- 保持默认模型为 `bailian/kimi-k2.5`
- 尽量少改配置，优先用证据驱动排障
- 没有稳定远程通道时，退化为输出用户可直接复制执行的本机命令

## 何时使用

- 用户直接说“救援mac”
- 用户说“本机 openclaw 没响应了”
- 用户说“Control UI 卡死 / 没有响应”
- 用户怀疑 `kimi-k2.5` 不再是默认模型
- 用户希望从 tx-2 协助排查本机 Mac 上的 OpenClaw

不要把这个 skill 用在：

- tx-2 自己的 OpenClaw 故障
- 非 OpenClaw 的 Docker、QingLong、反代或系统服务问题
- 需要大规模重装系统或清理用户数据的场景

## 默认策略

默认使用混合模式：

1. 先做标准探测，收集最小事实集
2. 如果环境里存在可用的 Mac SSH 通道，就远程执行探测与修复
3. 如果没有 SSH 通道，或 SSH 探测失败，就输出本机复制执行版命令

除非证据明确，不要：

- 重写 `~/.openclaw/openclaw.json`
- 删除 `paired.json`
- 清空会话、workspace 或 agents 目录
- 随意替换 provider 或默认模型

## 先收集这四个事实

无论本地执行还是经 SSH 包装执行，先拿到下面四项：

```bash
openclaw gateway status
lsof -nP -iTCP:18789 -sTCP:LISTEN
openclaw models status --plain
openclaw dashboard --no-open
```

如果需要验证“不是假在线”，再追加：

```bash
openclaw agent --to +10000000000 --message '请只回复OK，不要解释。' --json --timeout 60
```

## 远程执行分流

只有在下面环境变量都合理存在时，才尝试 SSH：

- `OPENCLAW_RESCUE_MAC_HOST`
- `OPENCLAW_RESCUE_MAC_USER`
- 可选：`OPENCLAW_RESCUE_MAC_PORT`

先读 `references/ssh-mode.md`，确认 SSH 包装方式和失败回退规则。

如果 SSH 不可用，不要卡住；直接把标准探测命令和修复命令整理成用户可复制执行的块。

## 证据驱动排障

按现象读对应 reference：

- `references/recovery-matrix.md`
  适用于大多数排障，尤其是 LaunchAgent 未加载、端口未监听、默认模型漂移
- `references/browser-session-cleanup.md`
  适用于 gateway 健康但旧网页仍“没响应”或日志里出现 `missing scope: operator.read`

优先级固定如下：

1. 先判断 gateway 是否存在且在监听
2. 再判断默认模型是否仍是 `bailian/kimi-k2.5`
3. 最后才判断是否为旧网页会话残留

不要在 gateway 未恢复前，把问题归因为前端缓存。

## 输出要求

最终答复必须包含这五项：

1. 根因判断
2. 关键证据
3. 已执行动作或建议执行动作
4. 修复后验证方式
5. 下一步建议

如果没有远程通道，要明确写明“以下命令需在本机 Mac 执行”。

## 成功判定

满足以下条件即可认为救援成功：

- `openclaw gateway status` 显示 `LaunchAgent (loaded)` 或等价健康状态
- `127.0.0.1:18789` 正在监听
- `openclaw models status --plain` 输出 `bailian/kimi-k2.5`
- `openclaw dashboard --no-open` 能给出新 dashboard URL
- 如做了实测，`openclaw agent ...` 能正常返回

## 参考文件

- 标准排障矩阵：`references/recovery-matrix.md`
- SSH 模式：`references/ssh-mode.md`
- 旧浏览器会话清理：`references/browser-session-cleanup.md`
