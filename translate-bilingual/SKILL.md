---
name: translate-bilingual
description: Use when the user wants to translate English content to Chinese and generate a bilingual side-by-side HTML page. Input can be a local file path, URL, or pasted text. Triggers on: 翻译, translate, 双栏, 中英对照, bilingual, side by side, 翻译并保存, 翻译页面, or when the user gives a file/URL wanting bilingual HTML output. Also triggers when user says 英译中, en2zh, 对照翻译.
---

# translate-bilingual: 中英对照翻译

给定本地文件、URL 或粘贴文本，将英文内容逐段翻译为中文，生成**双栏对照 HTML** 页面。

## 红线

1. 每段**英文原文和中文翻译必须成对出现**，在同一行对齐
2. 代码块、公式（`$...$` / `$$...$$`）、图片链接（`![alt](url)`）**保持原样不翻译**
3. Mermaid 流程图（````mermaid` 代码块）→ 使用 `<pre class="mermaid">` 渲染，引入 Mermaid JS 库动态渲染为 SVG
3. 专业术语根据上下文翻译，首次出现时在中文后括号标注英文原文
4. 禁止使用在线翻译 API——由你本身（Claude）直接翻译
5. 最终 HTML 必须是**自包含单文件**（所有 CSS inline，无外部依赖）

## 工作流

### Step 1：内容获取

根据用户输入类型获取内容：

```markdown
- URL 链接 → 使用 WebFetch 工具获取页面内容
- 本地文件路径 → 使用 Read 工具读取文件内容
- 粘贴文本 → 直接使用用户提供的文本
```

从内容中提取标题（第一个 `# ` 行或 `<h1>` 或页面 `<title>`）。

### Step 2：内容分块解析

将内容按以下类型分块，保留顺序：

| 块类型 | 检测规则 | 处理方式 |
|--------|----------|----------|
| 标题 | `# ` / `## ` / `### ` / `<h1>`~`<h4>` | 翻译标题文字，保留层级 |
| 段落 | 连续文本块（空行分隔） | 逐段翻译 |
| 代码块（普通） | ` ``` ` 包裹（非 mermaid）或 `<pre><code>` | 不翻译，原样保留 |
| Mermaid 流程图 | ` ```mermaid ` 包裹 | 不翻译，用 `<pre class="mermaid">` 渲染，引入 Mermaid JS |
| 引用块 | `> ` 开头或 `<blockquote>` | 翻译引用文字 |
| 列表 | `- ` / `* ` / `1. ` 开头或 `<ul>/<ol>` | 逐项翻译 |
| 表格 | `|` 分隔或 `<table>` | 翻译表头和单元格 |
| 图片 | `![alt](url)` 或 `<img>` | 不翻译，原样保留 |
| 公式 | `$...$` / `$$...$$` | 不翻译，原样保留 |
| 分割线 | `---` / `***` | 原样保留 |
| 空行 | 两个换行 | 保留间距 |

### Step 3：逐块翻译

对每个**非代码/非公式/非图片**的文本块：

1. **信**：逐句准确翻译，关键术语括号标注英文
2. **达**：调整语序使其符合中文表达习惯，拆分长句
3. **雅**：仅在需要时处理文化差异、双关语等

术语处理规则：
- 首次出现的专业术语：`"循环工程（Loop Engineering）"`
- 后续同术语：直接用中文
- 常见缩写（AI、API、MCP、CLI、CI/CD 等）保留英文

### Step 4：构建 HTML

构建自包含的 HTML 文件。结构如下：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{{标题}} · 中英对照</title>
  <style>
    /* 所有样式 inline 在此 */
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { font-family: ..., background: #fff; color: #1d1d1f; }
    .container { max-width: 960px; margin: 0 auto; padding: 24px; }
    .header { text-align: center; padding: 32px 0 16px; border-bottom: 1px solid #e8e8ed; }
    .header h1 { font-size: 1.8rem; color: #1d1d1f; }
    .header .meta { color: #86868b; font-size: 0.85rem; margin-top: 8px; }

    /* 双栏核心样式 */
    .pair { display: flex; gap: 24px; padding: 16px 0; border-bottom: 1px solid #f0f0f2; }
    .en, .zh { width: 50%; }
    .en { border-right: 1px solid #e8e8ed; padding-right: 20px; }
    .zh { padding-left: 20px; }
    .en p, .zh p { font-size: 0.95rem; line-height: 1.8; margin-bottom: 8px; }
    .en { color: #1d1d1f; }
    .zh { color: #515154; }

    /* 普通代码块（单列居中） */
    .code-single { width: 100%; }
    pre { background: #f5f5f7; border-radius: 8px; padding: 16px; overflow-x: auto; font-size: 0.85rem; }

    /* Mermaid 流程图容器 */
    .mermaid-wrap { width: 100%; padding: 20px 0; text-align: center; background: #fff; border-radius: 8px; border: 1px solid #e8e8ed; }

    /* 引用块 */
    blockquote { border-left: 4px solid #5b7fff; padding: 12px 20px; margin: 8px 0; background: #f8f9ff; }

    /* 标题 */
    h1 { font-size: 1.6rem; margin: 20px 0 8px; }
    h2 { font-size: 1.3rem; margin: 16px 0 6px; }
    h3 { font-size: 1.1rem; margin: 14px 0 6px; }
    h4 { font-size: 1rem; margin: 12px 0 4px; }

    /* 响应式 */
    @media (max-width: 768px) {
      .pair { flex-direction: column; gap: 0; }
      .en, .zh { width: 100%; }
      .en { border-right: none; border-bottom: 1px solid #e8e8ed; padding-right: 0; padding-bottom: 8px; }
      .zh { padding-left: 0; padding-top: 8px; }
    }

    /* 标签标记 */
    .tag { display: inline-block; font-size: 0.65rem; padding: 1px 6px; border-radius: 3px; margin-right: 4px; }
    .tag-en { background: #e3f2fd; color: #1565c0; }
    .tag-zh { background: #fff3e0; color: #e65100; }

    /* 页脚 */
    .footer { text-align: center; color: #86868b; font-size: 0.82rem; padding: 24px 0; border-top: 1px solid #e8e8ed; margin-top: 16px; }
  </style>
</head>
<body>
<div class="container">
  <div class="header">
    <h1>{{TITLE}}</h1>
    <p class="meta">{{SOURCE}}</p>
  </div>

  {{CONTENT}}

  <div class="footer">
    <p>{{SOURCE_FOOTER}} | 中英对照翻译 · 仅供学习参考</p>
  </div>
</div>
</body>
  <script>
    (function() {
      var m = document.createElement('script');
      m.src = 'https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.min.js';
      m.onload = function() {
        mermaid.initialize({ startOnLoad: true, theme: 'default', securityLevel: 'loose' });
        mermaid.run({ nodes: document.querySelectorAll('pre.mermaid') });
      };
      document.head.appendChild(m);
    })();
  </script>
</html>
```

#### 每种块类型的 HTML 渲染规则

**标题块**（`# 标题` → `<h1>` 等）：
```html
<div class="pair header-pair">
  <div class="en"><h2>Original Heading</h2></div>
  <div class="zh"><h2>原标题翻译</h2></div>
</div>
```

**段落块**（连续文本）：
```html
<div class="pair">
  <div class="en"><p>Original paragraph text...</p></div>
  <div class="zh"><p>翻译后的段落...</p></div>
</div>
```

**普通代码块**（不翻译，全宽单列）：
```html
<div class="pair">
  <div class="code-single" style="width:100%;">
    <pre><code>code content here</code></pre>
  </div>
</div>
```

**Mermaid 流程图**（不翻译，使用 Mermaid JS 渲染为 SVG）：
```html
<div class="pair">
  <div class="mermaid-wrap">
    <pre class="mermaid">
      flowchart LR
        A[Start] --> B[End]
    </pre>
  </div>
</div>
```

注意：将 ```mermaid 代码块内容放入 <pre class="mermaid"> 中，不转义箭头符号和方括号。Mermaid JS 会在页面加载后自动渲染。

**引用块**：
```html
<div class="pair">
  <div class="en"><blockquote><p>Original quote</p></blockquote></div>
  <div class="zh"><blockquote><p>引文翻译</p></blockquote></div>
</div>
```

**列表块**：
```html
<div class="pair">
  <div class="en"><ul><li>Item 1</li><li>Item 2</li></ul></div>
  <div class="zh"><ul><li>项目 1 翻译</li><li>项目 2 翻译</li></ul></div>
</div>
```

**图片**（不翻译，居中保留）：
```html
<div class="pair">
  <div style="width:100%;text-align:center;">
    <img src="url" alt="alt text" style="max-width:100%;">
  </div>
</div>
```

**公式**（不翻译，居中保留）：
```html
<div class="pair">
  <div style="width:100%;text-align:center;font-style:italic;">
    $$ formula $$
  </div>
</div>
```

**分割线**：
```html
<div class="pair">
  <div style="width:100%;"><hr></div>
</div>
```

### Step 5：保存文件

1. **文件名规则**：
   - 本地文件 `doc.md` → `doc.html`
   - URL `https://example.com/article` → `article.html`
   - 粘贴文本 → 用户指定或默认 `translation.html`
2. 使用 `Write` 工具写入到当前工作目录
3. 告知用户文件已保存的路径

## 输出示例结构

```
~/.claude/skills/translate-bilingual/
  SKILL.md
```

（无外部依赖，模板内嵌在工作流中）

## 验收标准

1. [ ] 中英双栏在同一行严格对齐
2. [ ] 代码块和公式完全保留，不翻译
3. [ ] 术语首次出现时中英对照
4. [ ] 移动端自动折叠为上下排列
5. [ ] 生成的文件是自包含 HTML，浏览器可直接打开
6. [ ] 原文档的标题层级正确映射到 HTML
