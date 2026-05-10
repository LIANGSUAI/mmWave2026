# Claude Paper Workflow 测试

- **创建时间**: 2026-05-10
- **Vault 路径**: D:/lqs/ob/mmWave2026

## 测试说明

此文件用于验证 Claude Code 论文阅读与 Obsidian 笔记工作流是否正常运行。

## 已完成的部署

1. 目录结构已创建（00_Inbox ~ 06_Reading_Queue, templates）
2. 论文笔记模板已创建：`templates/paper-note-template.md`
3. CLAUDE.md 工作流配置已写入
4. claude-defuddle skill 已安装到 `~/.claude/skills/defuddle/`

## 后续使用示例

### 搜索推荐论文
```
请围绕基于优化问题和密度分析的可解释恶意流量检测，搜索近三年相关论文，推荐 10 篇，按必读/可读/暂缓分类。先只给候选列表，不要生成笔记。
```

### 选择论文生成笔记
```
请阅读第 1、3、5 篇，分别生成 Obsidian 文献笔记，保存到 01_Papers/malicious-traffic-detection/。
```

### 生成综述
```
请基于 01_Papers/malicious-traffic-detection/ 下已有笔记，整理一份文献综述草稿，保存到 02_Literature_Reviews/。
```
