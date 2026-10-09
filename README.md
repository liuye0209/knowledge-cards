# make-knowledge-cards

把文章转换成 5~8 张知识卡片的 Agent Skill，符合 [Agent Skills 规范](https://agentskills.io)，可用于 OpenCode、Claude Code、Codex、Copilot CLI 等支持 `SKILL.md` 的工具。

## 解决什么问题

- **读完就忘**：长文章读完留不下可复习的要点，手工摘录成卡片又费时费力。
- **直接让 AI 总结不可靠**：常见失败包括——为凑数量硬拆知识点、编造原文没有的例子和背景知识、同一件事换个说法重复出现、一张卡塞进多个知识点。

本 Skill 用一份经过 8 轮真实测试（覆盖中文长文、英文文章、信息不足短文、重复内容文章、不支持输入等 5 类场景）的固定流程约束模型：先提取原文知识点，按知识点数量确定卡片数，逐卡填写，对照自查清单核对后才输出，得到忠实原文、可直接复习的卡片。

## 主要功能

- **两种输入**：直接粘贴文章全文，或本地 Markdown（`.md`）/ 纯文本（`.txt`）文件路径
- **固定卡片结构**：每张卡片包含**标题、核心知识、简明解释、例子或自测问题**四个字段
- **卡片数量由原文决定**：5~8 张是典型范围而非配额；不足 5 张时如实少出并附说明 `> 原文信息有限，仅生成 N 张卡片。`
- **一张卡只讲一个知识点**：不合并、不拆分、不重复
- **只写原文有的信息**：每个事实、数字、例子都能回溯到原文；原文没有合适例子时，改为出一道答案可从原文推出的自测题
- **输出前自查**：内置 5 项核对清单，先自查再输出
- **语言跟随请求**：用中文提问输出中文卡片，用英文提问输出英文卡片；请求没有文字描述时用原文语言
- **边界明确**：网页 URL（不抓取）、PDF、Anki 导出、图形界面暂不支持，会直接说明而非变通处理
- **标准结构**：由官方 `init_skill.py` 初始化，包含 `SKILL.md` 与 `agents/openai.yaml`，通过官方 `quick_validate.py` 校验

## 安装方法

Skill 本体是 `skills/make-knowledge-cards/` 目录（不含任何运行脚本）。将整个目录复制到你的 Agent 的 Skill 发现目录：

| 工具 | 项目级目录 | 全局（用户级）目录 |
| --- | --- | --- |
| OpenCode | `<项目>/.opencode/skills/` | `~/.config/opencode/skills/` |
| Claude Code | `<项目>/.claude/skills/` | `~/.claude/skills/` |
| Codex / Copilot CLI / Gemini CLI / VS Code | `<项目>/.agents/skills/` | `~/.agents/skills/` |

例如，在仓库根目录执行（Bash）：

```bash
mkdir -p .agents/skills
cp -r skills/make-knowledge-cards .agents/skills/
```

Windows PowerShell：

```powershell
New-Item -ItemType Directory -Force .agents\skills | Out-Null
Copy-Item -Recurse skills\make-knowledge-cards .agents\skills\
```

**OpenCode 免复制安装**：在项目的 `opencode.json(c)` 中把仓库的 `skills/` 目录注册为 Skill 源：

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "skills": ["./skills"]
}
```

Skill ID 为 `make-knowledge-cards`，重启会话后生效。

## 使用方法

安装后，在对话中直接对 Agent 说：

> 把这篇文章做成知识卡片：（粘贴文章全文）

或指定本地文件：

> 把 `notes/article.md` 做成知识卡片

Agent 会根据描述自动匹配并加载本 Skill；也可以在 OpenCode 中用 `@make-knowledge-cards` 显式触发。

**暂不支持**：网页 URL（不抓取网页）、PDF、Anki 导出、图形界面。向这些输入索要卡片时，Skill 会直接说明不支持并给出替代方式。

## 输入输出示例

### 示例 1：短文（信息不足，不凑数）

**输入**（`article.txt`）：

```text
番茄工作法由弗朗西斯科·西里洛在20世纪80年代末发明，使用25分钟工作加5分钟休息的循环。
```

**输出**：

```markdown
### 卡片 1：番茄工作法的发明者与起源

**核心知识**：番茄工作法由弗朗西斯科·西里洛在 20 世纪 80 年代末发明。

**简明解释**：这种时间管理方法并非由来已久的古老技巧，而是在 20 世纪 80 年代末由弗朗西斯科·西里洛这一具体人物提出的。

**例子/自测**：自测问题：番茄工作法是由谁、在什么时候发明的？答案：由弗朗西斯科·西里洛在 20 世纪 80 年代末发明。

### 卡片 2：番茄工作法的基本循环

**核心知识**：番茄工作法使用 25 分钟工作加 5 分钟休息的循环。

**简明解释**：该方法把时间切分成固定单元：一段 25 分钟的专注工作之后，紧接着一段 5 分钟的休息，两者构成一个不断重复的循环。

**例子/自测**：自测问题：一个循环包含多长的工作时间和多长的休息时间？答案：25 分钟工作，5 分钟休息。

> 原文信息有限，仅生成 2 张卡片。
```

原文只有一个知识点时输出 2 张卡并如实说明，而不是硬凑 5 张。

### 示例 2：长文章（8 张卡片，节选）

**输入**（`http-cache.md` 开头）：

```markdown
# HTTP 缓存机制详解

Web 性能优化中，HTTP 缓存是最重要的手段之一。合理利用缓存可以显著减少网络传输、降低服务器压力并加快页面加载速度。
```

**输出**（实际测试输出，8 张中的第 1 张）：

```markdown
### 卡片 1：强缓存的执行行为

**核心知识**：强缓存是指在缓存有效期内，浏览器直接使用本地缓存副本，不向服务器发送任何请求。

**简明解释**：强缓存是 Web 性能优化中 HTTP 缓存的重要方式之一，它让浏览器无需与服务器交互即可复用已缓存的资源，从而减少网络传输、降低服务器压力并加快页面加载速度。

**例子/自测**：自测：处于强缓存有效期内的资源，浏览器是向服务器发请求取回，还是直接用本地副本？答案：直接使用本地副本，不向服务器发送任何请求。
```

……其余 7 张分别讲 Expires 与 Cache-Control、协商缓存、Last-Modified、ETag、no-cache 与 no-store、缓存位置、缓存策略实践，一卡一点、互不重复。

### 示例 3：不支持的输入（正确拒绝）

**输入**：

> 请帮我把 https://example.com/news/2026/tech-article 这篇文章做成知识卡片

**输出**：

```markdown
**暂不支持**：该 Skill 不支持网页 URL 输入（不抓取网页）。

请改用以下任一方式提供文章内容：

- 直接把文章全文粘贴到对话中
- 提供本地 `.md` 或 `.txt` 文件路径
```

## 目录结构

```
skills/make-knowledge-cards/
├── SKILL.md            # Skill 主体（元数据 + 指令）
└── agents/openai.yaml  # 界面展示元数据
```

## 验证

使用官方 `quick_validate.py`（来自 [openai/skills](https://github.com/openai/skills) 的 skill-creator）：

```bash
python quick_validate.py skills/make-knowledge-cards
# Skill is valid!
```

## 许可证

[MIT](LICENSE)
