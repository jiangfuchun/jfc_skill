# jfc_skill

JFC Claude Code 技能仓库 — 为 Claude Code 提供专业化的行业研究与翻译能力。

## 技能列表

| Skill | 描述 | 触发关键词 |
|-------|------|-----------|
| **company-research** | 上市公司深度研报引擎。整合 8 大经典方法论（波特五力、护城河、杜邦分解、估值矩阵、SWOT、ESG、管理层评估、行业生命周期），生成结构化的 HTML 研报 | `公司研报` `深度研究` `分析公司` `写研报` `equity research` `股票分析` `上市公司研报` |
| **industry-research** | 行业深度研报引擎。输入一个行业名词，自动完成全行业研究分析 — 涵盖产业链拆解、全球竞争格局、龙头公司分析、A 股标的扫描、投资价值评估，输出精美 HTML 研报 | `行业研报` `industry research` `行业深度研究` `产业链分析` `行业分析` `行业报告` |
| **translate-bilingual** | 中英对照翻译工具。将英文内容（支持本地文件、URL、粘贴文本）逐段翻译为中文，生成双栏对照 HTML 页面，支持代码块、Mermaid 流程图、公式等元素的保留 | `翻译` `translate` `双栏` `中英对照` `bilingual` `英译中` `en2zh` |

## 安装方法

### 方式一：直接克隆（推荐）

```bash
# 克隆仓库到本地
git clone https://github.com/jiangfuchun/jfc_skill.git ~/.claude/skills/

# 或在 Claude Code 中直接使用 /skill 命令加载
# Claude Code → Settings → Skills → Add → 选择对应 skill 目录
```

### 方式二：按需安装单个 Skill

将对应 Skill 目录放置在 Claude Code 的 skills 加载路径中：

```bash
# 安装 company-research
cp -r company-research ~/.claude/skills/

# 安装 industry-research
cp -r industry-research ~/.claude/skills/

# 安装 translate-bilingual
cp -r translate-bilingual ~/.claude/skills/
```

### 方式三：在 Claude Code 项目配置中引用

在项目的 `.claude/settings.json` 中添加 skills 路径：

```json
{
  "skills": {
    "paths": ["/path/to/jfc_skill/company-research", "/path/to/jfc_skill/industry-research", "/path/to/jfc_skill/translate-bilingual"]
  }
}
```

### 使用方式

安装完成后，在 Claude Code 会话中直接输入对应触发关键词即可自动调用对应的 Skill。例如：

- 「**帮我看一下特斯拉的深度研报**」→ 触发 `company-research`
- 「**写一份液冷行业的深度报告**」→ 触发 `industry-research`
- 「**把这个文档翻译成中英对照**」→ 触发 `translate-bilingual`
