---
name: industry-research
description: Use when the user provides a single industry name (Chinese or English) and wants a comprehensive industry deep research report in HTML format. Triggers on: 行业研报, industry research, 行业深度研究, 产业链分析, 行业分析, industry analysis, 写行业研报, 行业报告. Input is a single industry name only.
---

# industry-research: 行业深度研报引擎

输入一个**行业名词**（如"液冷"、"AI芯片"、"人形机器人"、"光伏"、"储能"），自动完成全行业研究分析，生成精美 HTML 行业深度研报。

## 分析方法论

整合 7 大经典分析方法，按行业特征组合使用：

| 方法论 | 适用环节 | 主要用途 |
|--------|----------|----------|
| 行业生命周期 | Step 1 | 定位 S 曲线位置 |
| 价值链分析 | Step 2 | 产业链拆解 |
| 波特五力 | Step 3 | 竞争格局评估 |
| 晨星护城河 | Step 4 | 龙头公司壁垒分析 |
| 杜邦分解 | Step 5-6 | 财务健康度评估 |
| 估值矩阵 | Step 6 | 投资价值判断 |
| SWOT | Step 7 | 总成收束 |

## 红线

1. **所有判断必须附证据或标注"待补充"**，禁止"行业前景广阔"等空话
2. 不回避负面，风险清单必须量化（发生概率 × 影响程度）
3. 投资建议三选一：**推荐 / 中性 / 回避**
4. 禁止出现的词汇：赛道很大、团队优秀、前景广阔、值得期待、任重道远、未来可期
5. 数据取自搜索结果的权威信源（行业白皮书、咨询公司报告、上市公司财报）
6. 最终 HTML 必须是**自包含单文件**（所有 CSS inline，无外部依赖）

## 工作流

### Step 0：信息采集（AnySearch + batch_search）

使用 `anysearch` skill 的 `batch_search` 命令进行信息采集。并发 5-7 个查询方向：

```bash
# 查询方向 1：行业规模与增长
python3 <skill_dir>/scripts/anysearch_cli.py search "<行业> 行业 市场规模 2025 2026 CAGR 预测" --domain business --sub_domain industry_report --zone cn --max_results 10

# 查询方向 2：行业规模（国际视角）
python3 <skill_dir>/scripts/anysearch_cli.py search "<industry> market size 2025 2026 CAGR forecast" --domain business --sub_domain market_report --zone intl --max_results 10

# 查询方向 3：产业链分析
python3 <skill_dir>/scripts/anysearch_cli.py search "<行业> 产业链 上中下游 分析 结构" --max_results 10

# 查询方向 4：竞争格局
python3 <skill_dir>/scripts/anysearch_cli.py search "<行业> 竞争格局 市场份额 排名 龙头" --domain business --sub_domain competition --zone cn --max_results 10

# 查询方向 5：全球龙头
python3 <skill_dir>/scripts/anysearch_cli.py search "<行业> 全球 龙头 TOP5 企业 排名 份额" --max_results 10

# 查询方向 6：A 股相关
python3 <skill_dir>/scripts/anysearch_cli.py search "<行业> 概念股 龙头股 A股 上市公司 标的" --domain business --sub_domain industry_report --zone cn --max_results 10

# 查询方向 7：技术趋势与政策
python3 <skill_dir>/scripts/anysearch_cli.py search "<行业> 技术趋势 发展方向 政策 规划 2026" --domain business --sub_domain industry --zone cn --max_results 10
```

用 `batch_search` 合并为一次 API 调用（减少延迟）。如果某个方向无结果，标注"待补充"。

### Step 1：行业生命周期定位

整合搜索到的行业规模数据，判断行业所处的 S 曲线位置：

| 阶段 | 渗透率 | 增长特征 | 竞争特征 | 适用估值 |
|------|--------|----------|----------|----------|
| 导入期 | <5% | 高研发投入，尚未盈利 | 技术路线未收敛 | PS/用户估值 |
| 成长期 | 5-30% | 营收高速增长 | 份额争夺，龙头初现 | PEG/PS/DCF |
| 成熟期 | >30% | 增速放缓至GDP+ | 格局稳定，存量博弈 | PE/DCF/EV/EBITDA |
| 衰退期 | 峰值已过 | 负增长 | 退出整合 | PB/清算估值 |

**输出要点**：
- 当前全球市场规模（$XX亿）及中国市场规模（¥XX亿）
- 未来 3-5 年 CAGR（引用权威来源）
- 渗透率位置及判断依据
- 核心驱动因子（政策/技术/需求/替代）

### Step 2：产业链深度分析

绘制 `上游 → 中游 → 下游 → 终端应用` 链条结构：

**分析框架（每个环节）：**
- 环节名称与定义
- 核心产品/技术/服务
- 全球代表企业及中国代表企业
- 毛利率/利润率区间（如有数据）
- 竞争格局（集中/分散/寡头）
- 进入壁垒（高/中/低）
- 价值量占比（该环节在产业链利润中的占比）

**推荐表格式输出（HTML 中将渲染为卡片）**：

```
| 环节 | 核心内容 | 全球代表 | 中国代表 | 毛利率 | 壁垒 |
|------|----------|----------|----------|--------|------|
| 上游 | ... | ... | ... | ... | ... |
| 中游 | ... | ... | ... | ... | ... |
| 下游 | ... | ... | ... | ... | ... |
```

产业链每个环节的值占比用纯 CSS 水平条形图展示（`.bar-fill` + 分段颜色）。

### Step 3：全球竞争格局（波特五力 + 份额矩阵）

#### 3.1 波特五力

| 维度 | 判断 | 依据 |
|------|------|------|
| 现有竞争强度 | 高/中/低 | CR3/CR5，产品差异化程度，行业增速 |
| 潜在进入者 | 高/中/低 | 技术门槛、资本门槛、政策壁垒 |
| 替代品威胁 | 高/中/低 | 替代品性价比、转换成本 |
| 供应商议价力 | 高/中/低 | 供应商集中度、单一来源风险 |
| 买方议价力 | 高/中/低 | 客户集中度、转换成本 |

**综合评级**：竞争格局好 / 中等 / 恶劣

#### 3.2 市场份额矩阵

Top 5 企业 + 市占率，用纯 CSS 水平堆叠条展示：

```html
<div class="share-bar">
  <div class="share-segment" style="width:XX%;background:#...">企业A XX%</div>
  ...
  <div class="share-segment share-other" style="width:XX%;">其他 XX%</div>
</div>
```

### Step 4：全球龙头公司深度分析

选取 3-5 家全球龙头，每家：

- **公司卡片**（`.leader-card`）：
  - 公司名 + 总部所在地 + 上市地 + 市值
  - 主营业务与该行业的关联
  - 核心护城河（晨星五来源分析）
  - 近 3 年营收与净利润趋势
  - 市场份额与排名
  - 战略动向

用 `grid-2` 布局，两家一行。如需强调某家使用 `.alert-green` 高亮框。

### Step 5：A 股标的扫描

按产业链环节分组列出 A 股相关上市公司：

| 代码 | 名称 | 产业链环节 | 核心业务 | 市值(亿) | 营收(亿) | PE | 毛利率 | 评级 |
|------|------|-----------|----------|---------|---------|------|--------|------|
| ... | ... | ... | ... | ... | ... | ... | ... | ... |

**评级标签**：`<span class="tag-core">核心</span>` / `<span class="tag-ok">关注</span>` / `<span class="tag-mute">观察</span>`

数量：5-15 家，覆盖产业链各环节。

### Step 6：投资价值分析

#### 6.1 财务筛选重点

从 Step 5 的标的中选取 3-5 家最值得关注的公司，从以下维度分析：

| 维度 | 指标 | 筛选标准 |
|------|------|----------|
| 成长性 | 营收增速(3年) | >20% 优秀 / 10-20% 良好 |
| 盈利质量 | 净利率 / 毛利率趋势 | 稳定或提升为优 |
| 资本回报 | ROE / ROIC | >10% 及格 / >15% 优秀 |
| 财务安全 | 经营现金流/净利润 | >1.0 健康 / >0.7 关注 |
| 估值 | PE(TTM) / PEG | PEG<1 低估 / 1-2 合理 |

#### 6.2 估值矩阵（三情景）

对每家推荐公司：

```html
<div class="price-card">
  <h4>{{公司名}} - 三情景目标价</h4>
  <div class="price-table">
    <div class="price-item price-conservative">
      <div class="scenario">保守</div>
      <div class="target">XX元</div>
      <div class="condition">XX情景假设</div>
    </div>
    <div class="price-item price-neutral">
      <div class="scenario">中性</div>
      <div class="target">XX元</div>
      <div class="condition">XX情景假设</div>
    </div>
    <div class="price-item price-optimistic">
      <div class="scenario">乐观</div>
      <div class="target">XX元</div>
      <div class="condition">XX情景假设</div>
    </div>
  </div>
</div>
```

#### 6.3 推荐排序与投资逻辑

按推荐优先级排列，每家一段"投资逻辑"（100-200字），说明：
- 为什么值得投资（核心逻辑）
- 什么情况下判断错误（容错机制）

### Step 7：风险与展望（SWOT 总成）

#### SWOT 四格

| S（优势） | W（劣势） |
|-----------|-----------|
| 3-5 条内部优势 | 3-5 条内部劣势 |

| O（机会） | T（威胁） |
|-----------|-----------|
| 3-5 条外部机会 | 3-5 条外部威胁 |

每条不超过 20 字。

#### 风险清单

| 风险 | 概率 | 影响 | 描述 |
|------|:----:|:----:|------|
| 政策风险 | 高/中/低 | 高/中/低 | XXX |
| 技术风险 | 高/中/低 | 高/中/低 | XXX |
| 竞争风险 | 高/中/低 | 高/中/低 | XXX |
| 需求风险 | 高/中/低 | 高/中/低 | XXX |

### Step 8：构建 HTML

构建自包含的 HTML 文件。使用与 `company-research` 一致的 CSS 组件体系，并增强以下自定义组件：

#### CSS 组件清单

**基础**：* reset, body, 字体, `.container` (max-width:1100px), `.section` (白色圆角卡片)

**Hero 区**：`.hero` (深蓝渐变) + `.hero::after` (光晕条) + `.code` (行业标签) + `.gradient` 文字 + `.subtitle` + `.meta`

**KPI 行**：`.kpi-row` (4列) + `.kpi-card` + `.kpi-num` + `.kpi-label`

**数据可视化**：
- `.share-bar` (市场份额水平堆叠条) + `.share-segment` (分段)
- `.chain-grid` (产业链4列卡片) + `.chain-card` + `.chain-arrow` (箭头分隔) + 每条 `.chain-card` 顶部色条区分环节
- `.big-number` (大号数字 callout) + `.big-number .label`

**表格**：`table` + `thead th` + `tbody td` + `tr:hover`

**标签**：`.tag-core` (绿) / `.tag-warn` (红) / `.tag-ok` (蓝) / `.tag-mute` (灰) / `.tag-hot` (橙）

**卡片**：`.section` (白色) / `.leader-card` (龙头公司, grid-2) / `.highlight` (蓝绿高亮)

**警示框**：`.alert-yellow` / `.alert-red` / `.alert-green`

**SWOT**：`.swot-grid` (2×2) + `.swot-s` (绿) / `.swot-w` (红) / `.swot-o` (蓝) / `.swot-t` (橙)

**目标价**：`.price-card` (深色背景) + `.price-table` (三栏) + `.price-item`

**页脚**：`.disclaimer`

**响应式**：`@media (max-width: 768px)` — Hero font缩小, KPI变2列, grid变1列, 表格字体缩小, padding减少

#### 文件命名

```
{行业名词}_行业深度研报_YYYYMMDD.html
```

输出到当前工作目录（使用 `Write` 工具）。

## 输出章节结构

```
一、行业概览与生命周期定位
  市场规模 / CAGR / 渗透率 / S 曲线位置 / 核心驱动因子

二、产业链深度分析
  上游 → 中游 → 下游 → 终端
  每环节代表企业 / 壁垒 / 毛利率 / 价值占比

三、全球竞争格局
  波特五力 / 市场份额矩阵 / CR3/CR5

四、全球龙头公司分析
  3-5 家深度分析（护城河 / 财务 / 战略）

五、A 股标的扫描
  5-15 家上市公司，按产业链分组

六、投资价值分析
  3-5 家重点推荐 / 三情景估值 / 投资逻辑

七、风险与展望（SWOT）
  SWOT 四格 / 风险清单

数据来源与免责声明
```

## 验收标准

1. [ ] 输入仅一个行业名词 → 输出完整研报
2. [ ] 产业链上中下游清晰完整
3. [ ] 全球市场数据和龙头公司覆盖
4. [ ] A 股标的信息完整
5. [ ] 投资建议有明确逻辑和估值支撑
6. [ ] HTML 格式精美，重点数据高亮
7. [ ] 所有数据标注来源，空缺处标注"待补充"

## 输出示例

```
~/.claude/skills/industry-research/
  SKILL.md
```
