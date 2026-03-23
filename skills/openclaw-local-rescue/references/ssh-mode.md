# SSH 模式

这个 skill 默认不假设 tx-2 一定能直连用户的 Mac。

只有在下面环境变量已配置时，才尝试远程执行：

- `OPENCLAW_RESCUE_MAC_HOST`
- `OPENCLAW_RESCUE_MAC_USER`
- 可选：`OPENCLAW_RESCUE_MAC_PORT`

## SSH 包装方式

端口默认 `22`：

```bash
SSH_PORT="${OPENCLAW_RESCUE_MAC_PORT:-22}"
SSH_TARGET="${OPENCLAW_RESCUE_MAC_USER}@${OPENCLAW_RESCUE_MAC_HOST}"
ssh -p "$SSH_PORT" "$SSH_TARGET" 'openclaw gateway status'
```

标准探测可分别包装执行：

```bash
ssh -p "$SSH_PORT" "$SSH_TARGET" 'openclaw gateway status'
ssh -p "$SSH_PORT" "$SSH_TARGET" 'lsof -nP -iTCP:18789 -sTCP:LISTEN'
ssh -p "$SSH_PORT" "$SSH_TARGET" 'openclaw models status --plain'
ssh -p "$SSH_PORT" "$SSH_TARGET" 'openclaw dashboard --no-open'
```

## 先做连通性探测

不要一上来就执行修复命令。先确认 SSH 能通：

```bash
ssh -o BatchMode=yes -o ConnectTimeout=5 -p "$SSH_PORT" "$SSH_TARGET" 'echo ok'
```

如果这一步失败：

- 明确告诉用户 tx-2 到本机 Mac 没有可用远程通道
- 切换为“以下命令请在本机 Mac 执行”的模式
- 继续输出完整排障步骤，不要中止任务

## 远程修复示例

当证据显示 LaunchAgent 未加载时：

```bash
ssh -p "$SSH_PORT" "$SSH_TARGET" 'openclaw gateway install --force'
ssh -p "$SSH_PORT" "$SSH_TARGET" 'openclaw gateway status'
ssh -p "$SSH_PORT" "$SSH_TARGET" 'lsof -nP -iTCP:18789 -sTCP:LISTEN'
```

当证据显示默认模型漂移时：

```bash
ssh -p "$SSH_PORT" "$SSH_TARGET" 'openclaw models set bailian/kimi-k2.5'
ssh -p "$SSH_PORT" "$SSH_TARGET" 'openclaw models status --plain'
```

## 输出要求

在 SSH 模式下，对用户要明确说明：

- 哪些命令已由 tx-2 远程执行
- 哪些结论来自远程回读
- 如果 SSH 失败，已经自动回退到手动模式

不要把“SSH 不通”误判为 “OpenClaw 已经坏了”。
