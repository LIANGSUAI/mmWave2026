# Obsidian Vault — mmWave2026

当前目录即为 Obsidian vault 根目录。

## 论文阅读与 Obsidian 笔记工作流

### 目录结构

- `00_Inbox/` — 临时收件箱、测试文件
- `01_Papers/` — 论文笔记（默认保存位置）
- `02_Literature_Reviews/` — 文献综述
- `03_Experiments/` — 实验记录
- `04_Thesis/` — 毕业论文材料
- `05_References/` — 参考文献管理
- `06_Reading_Queue/` — 候选论文清单
- `templates/` — 笔记模板

### 研究方向

基于优化问题和密度分析的可解释恶意流量检测研究。

### 工作流规则

1. 所有论文笔记直接保存为 Markdown 文件。
2. 默认论文笔记保存到 `01_Papers/`。
3. 候选论文清单保存到 `06_Reading_Queue/`。
4. 文献综述保存到 `02_Literature_Reviews/`。
5. 实验记录保存到 `03_Experiments/`。
6. 毕业论文材料保存到 `04_Thesis/`。
7. 论文信息必须真实可查，尽量包含 DOI / arXiv / venue / year / URL。
8. 不允许编造文献；不确定内容标记为"待核验"。
9. 生成笔记前先让用户选择论文，不要自动批量下载。
10. 保存笔记后提醒用户是否需要 git add / commit / push。

### 使用流程

**第一步 — 搜索推荐论文：**
> "请围绕 [研究主题]，搜索近三年相关论文，推荐 10 篇，按必读/可读/暂缓分类。先只给候选列表，不要生成笔记。"

**第二步 — 选择论文生成笔记：**
> "请阅读第 1、3、5 篇，分别生成 Obsidian 文献笔记，保存到 01_Papers/[子目录]/。"

**第三步 — 生成综述：**
> "请基于 01_Papers/[子目录]/ 下已有笔记，整理一份文献综述草稿，保存到 02_Literature_Reviews/。"

### 笔记模板

论文笔记使用模板：`templates/paper-note-template.md`
