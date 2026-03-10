# Trae Skills Collection

个人收集的 Trae AI Skills。

## Skills 列表

| Skill | 描述 |
|-------|------|
| [feishu-isolation-setup](./skills/feishu-isolation-setup/) | 飞书机器人多用户隔离配置（会话隔离、记忆隔离） |

## 使用方法

将 skill 目录复制到你的项目 `.trae/skills/` 目录下：

```bash
cp -r skills/feishu-isolation-setup /path/to/your/project/.trae/skills/
```

或者克隆整个仓库：

```bash
git clone https://github.com/cgaof/trae-skills.git
cp -r trae-skills/skills/* /path/to/your/project/.trae/skills/
```

## Skill 说明

### feishu-isolation-setup

为飞书机器人配置多用户隔离，实现：
- **会话隔离**：每个用户的对话历史独立
- **记忆隔离**：每个用户存储的信息独立
- **Cron 隔离**：定时任务在独立上下文中执行

适用于飞书 bot 设置为全公司可用的场景。

## 贡献

欢迎提交 Issue 和 Pull Request！

## 许可证

MIT License
