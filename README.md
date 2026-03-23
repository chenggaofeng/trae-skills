# Trae Skills Collection

个人收集的 Trae AI Skills。

## Skills 列表

| Skill | 描述 |
|-------|------|
| [feishu-isolation-setup](./skills/feishu-isolation-setup/) | 飞书机器人多用户隔离配置（会话隔离、记忆隔离） |
| [skill-sync-to-github](./skills/skill-sync-to-github/) | 将本地 skill 同步到 GitHub 仓库 |
| [npm-to-docker](./skills/npm-to-docker/) | 将npm全局安装的应用容器化为Docker镜像 |
| [openclaw-local-rescue](./skills/openclaw-local-rescue/) | 从 tx-2 辅助诊断和救援本机 OpenClaw，优先恢复 gateway 与 Kimi 2.5 |

## 使用方法

将 skill 目录复制到你的项目 `.trae/skills/` 目录下：

```bash
cp -r skills/feishu-isolation-setup /path/to/your/project/.trae/skills/
```

或者克隆整个仓库：

```bash
git clone https://github.com/chenggaofeng/trae-skills.git
cp -r trae-skills/skills/* /path/to/your/project/.trae/skills/
```

对于支持 GitHub 路径安装的 Skill Installer，也可以直接安装单个 skill：

```bash
scripts/install-skill-from-github.py --repo chenggaofeng/trae-skills --path skills/openclaw-local-rescue
```

## Skill 说明

### feishu-isolation-setup

为飞书机器人配置多用户隔离，实现：
- **会话隔离**：每个用户的对话历史独立
- **记忆隔离**：每个用户存储的信息独立
- **Cron 隔离**：定时任务在独立上下文中执行

适用于飞书 bot 设置为全公司可用的场景。

### skill-sync-to-github

将本地创建或更新的 skill 同步到 GitHub 仓库 `trae-skills`。

功能：
- 复制 skill 到 Git 仓库
- 提交并推送到 GitHub
- 更新 README.md

适用于用户创建新 skill 或更新现有 skill 后需要同步到 GitHub 的场景。

### openclaw-local-rescue

从 tx-2 或其他辅助环境，快速诊断和救援本机 OpenClaw。

功能：
- 标准化检查 gateway、监听端口、默认模型和 dashboard
- 优先恢复 `bailian/kimi-k2.5` 为默认模型
- 在没有 SSH 通道时退化为本机复制执行步骤
- 区分后端故障与旧浏览器会话残留

## 贡献

欢迎提交 Issue 和 Pull Request！

## 许可证

MIT License
