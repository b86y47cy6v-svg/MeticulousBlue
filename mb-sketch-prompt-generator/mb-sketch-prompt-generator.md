---
name: sketch-prompt-generator
description: |
  从文本中提取核心概念，每个概念生成一段独立的手绘视觉笔记图片提示词。融合两套提示词体系——A：自然语言设计简报（牛皮纸/做旧白纸、手绘墨水线条、荧光笔强调、咖啡渍折纸边）与 B：MJ/SD 关键词体系（Sketchnote / Marker Pen / Cross-Hatching / 技术参数）——为每个核心概念逐一决策纸张、色调、图标和强调方式，最终输出 N 段独立英文提示词。
  触发方式：/sketch、/sketch-prompt-generator、生成插图、视觉笔记、手绘笔记、帮我给这段文字生成一张图、画一个sketchnote、用AI建立知识库配图
  Extract core concepts from text and generate one independent hand-drawn sketchnote image prompt per concept.
  Trigger: /sketch, "generate a sketchnote for this", "make visual notes from this text"
---

# sketch-prompt-generator：手绘视觉笔记提示词生成器（一文多图）

## 用途

从用户提供的文本中提取核心概念，为**每个概念**生成一段独立、可直接投入 Midjourney / Stable Diffusion / DALL-E 使用的英文手绘视觉笔记图片提示词。

输入：一段文字  
输出：1 张结构总览图 + N 张观点图 × N 段独立提示词

---

## 前置条件

- 用户已提供文字内容
- 如用户未提供任何文本 → 提示：「请给我一段文字内容。我会提取核心概念，为每个概念生成一段手绘视觉笔记的图片提示词。」
- 本 skill 只生成**提示词**，不生成图片

---

## 核心原则

1. **一张结构总览图 + N 张结构观点图**。每张图都是结构型构图——总览图展示全篇逻辑骨架，观点图深挖单个论点的内部拆解（子要素+关系）。没有"金句卡片"式的 Single Spotlight 图。
2. **观点图也是微骨架**。不像总览图概括整篇，但每个观点图把该论点的 2-4 个子要素及其关系画清楚——看完一张图就掌握这个观点的完整逻辑。
3. **中文是主角，图标是配角**。画面以中文大标题 + 中文节点标签为核心视觉元素。手绘图标作为辅助。中文即使渲染不完美也必须在画面上。
4. **模型中文渲染的实际情况**：越短的词成功率越高。主标题 4-8 字中文，节点标签 2-4 字中文。不写段落/正文。加上引导词让模型将中文视为手绘艺术元素。

---

## 执行流程

---

### Phase 0：输入校验

| 情况 | 处理 |
|:---|:---|
| 无文本 | 提示用户提供文字 |
| 极短文（<50 字）| 全文即一个核心概念，输出**1 段**提示词 |
| 正常长度（50-2000 字）| 标准流程，提取 N 个核心概念 |
| 超长文（>2000 字）| 征求用户：提取全文核心概念还是指定段落？全文模式提取 5-8 个概念 |

**检查点**：✅ 输入校验完成，概念数量范围已确定

---

### Phase 0.5：提取全篇结构总览主题

**目标**：从整篇文本中提炼出内容的逻辑骨架——不是"这篇在讲什么主题"，而是"这个主题的**内部结构和逻辑脉络**是什么"。结构总览图是一张心智地图，让观众在没看具体观点之前，先掌握整篇的框架。

#### 提炼规则

1. **识别内容的结构类型**：流程型（先A→再B→最后C）、层级型（上层X→中层Y→底层Z）、模块型（三个并列要素）
2. **结构总览主题是一句完整陈述**，概括全文的逻辑骨架
3. **"点"要有，但"点和点之间的箭头/连线"才是核心**——结构总览图的价值在关系，不在单个节点
4. **主标题**：4-6 字，概括整篇在做什么事
5. **节点标签**：每个结构节点 2-4 字中文，放在对应的图标旁边

#### 结构类型判断

| 内容特征 | 结构类型 | 构图选择 | 示例 |
|:---|:---|:---|:---|
| 有先后步骤、「第一步…第二步…」 | 流程型 | Flow Pipeline | 搭建知识库的5个步骤 |
| 有层级关系、「上层…底层…」 | 层级型 | Stacked Layers | Karpathy三层架构 |
| 多个并列要素围绕一个中心 | 模块型 | Hub & Spokes | 知识库的三个好处 |
| 有输入→处理→输出的闭环 | 闭环型 | Circular Flow | 学习飞轮 |
| 从问题到方案的递进 | 路径型 | Left-to-Right Journey | 从零散知识到第二大脑 |

#### 输出格式

向用户展示，确认后再继续：

```
## 🗺 全篇结构总览

**总览主题**：{一句自包含的陈述，概括整篇的逻辑骨架}
**结构类型**：{流程型/层级型/模块型/闭环型/路径型}
**中文主标题**：{4-6 字，如「搭建专属知识库」「三层架构图」}
**结构节点**：
  ① {节点1标签 2-4字} — {简述}
  ② {节点2标签 2-4字} — {简述}
  ③ ...
**构图**：{Flow Pipeline / Stacked Layers / Hub & Spokes / Circular Flow / Left-to-Right Journey}
**色调**：{温暖/冷静/活力}

确认无误后继续提取具体观点。
```

**检查点**：✅ 结构总览已确认 ✅ 结构类型匹配内容逻辑 ✅ 节点标签 2-4 字

---

### Phase 1：提取核心概念

**目标**：从文本中萃取出值得独立图示化的 N 个核心观点，每个都是一句自包含的判断。

#### 提取规则

1. **每个概念是一句自包含的陈述**——不看原文也能懂它在说什么
2. **优先提取有视觉锚点的概念**——包含具体名词/动作/隐喻的优先
3. **每个概念自成一个单元**——不依赖其他概念即可成立
4. **数量**：短文 1-3 个，正常文 3-5 个，长文 5-8 个
5. **不可图标化的概念跳过**——纯抽象且无隐喻的概念（如「认知升级的本质是打破框架」极度抽象），用最接近的隐喻翻译

#### 输出格式

向用户展示提炼结果，让用户确认后再继续：

```
## 📖 核心概念提取

从原文中提取了 {N} 个核心概念：

| # | 概念（一句话） | 视觉锚点 | 可图标化 |
|:---:|:---|:---|:---:|
| 1 | {自包含的一句话} | {具体名词/动作} | ✅ / ⚠️隐喻 |
| 2 | {同上} | {同上} | ... |

如需增删或调整某个概念，请告诉我。否则我将为以上 {N} 个概念各自生成一段提示词。
```

**检查点**：✅ 用户确认概念列表 ✅ 每个概念都是自包含的一句话

---

### Phase 2：逐观点微骨架拆解

**目标**：为 Phase 1 的每个概念，拆解出 2-4 个子要素及其关系——每个观点图都是一张微骨架图，不是金句卡片。

#### 核心变化

**观点图不再使用 Single Spotlight。** 每个观点必须拆解出子要素和关系——对比、因果、步骤、层级、并列。单论点但无结构的观点，要么找到隐藏结构，要么合并到其他观点中。

#### 每个观点的决策维度

| 决策 | 依据 | 选项 |
|:---|:---|:---|
| **纸张** | 概念的情感温度 | 温暖/个人/日常 → Kraft paper / 冷静/精确/商务 → Aged white paper |
| **点缀色** | 概念所属的域 | 健康→绿色 / 技术→蓝色 / 商业→红色 / 学习→黄色 / 创意→多彩 / 默认→红黄蓝 |
| **子要素** | 概念的内部逻辑 | 2-4 个子要素，每个 2-4 字中文标签 + 一个小图标 |
| **关系类型** | 子要素之间的逻辑 | 见下表 |
| **构图** | 关系类型决定 | 见下表 |

#### 关系类型与构图

| 观点的内在逻辑 | 关系类型 | 构图 | 描述 |
|:---|:---|:---|:---|
| 怎么做、步骤、操作流程 | Steps | Mini Pipeline | 2-4 个子要素横向排列，箭头连接，如「问AI→它搜集→它学完→它教你」 |
| 对比、不是A而是B、新旧对照 | Contrast | Split Card | 左右或上下分两半，中间分隔线，两边各一个图标+标签 |
| 因果、因为A所以B、触发条件 | Cause-Effect | Arrow Flow | 左边原因→右边结果，箭头连接 |
| 并列要素、组成部分 | Components | Trio/Grid | 2-4 个图标均匀分布，各自带标签，无箭头 |
| 层级、从浅到深、从基础到高级 | Hierarchy | Stacked Layers | 从下往上叠或从上往下流 |
| 闭环、循环、持续迭代 | Loop | Circular Flow | 3-4 个节点环形排列，循环箭头连接 |
| 问题→方案、困境→解法 | Problem-Solution | Before/After Split | 左边问题状态→右边解决状态 |

**注意**：每个观点只用一种关系类型。**禁止 Single Spotlight**——如果一个观点拆不出至少 2 个子要素，说明它不是一个独立观点，应该合并。

#### 输出格式

对每个观点展示微骨架决策摘要：

```
## 🎨 视觉方案

### 观点 1：{一句话}

- **纸张**：{Kraft / Aged White}
- **色板**：大地基调 + {点缀色}
- **关系类型**：{Steps / Contrast / Cause-Effect / Components / Hierarchy / Loop / Problem-Solution}
- **构图**：{对应构图}
- **中文主标题**：「{4-8字}」
- **子要素**：
  · 「{标签1 2-4字}」— {简述+图标}
  · 「{标签2 2-4字}」— {简述+图标}
  · ...

### 观点 2：{一句话}
...
```

**检查点**：✅ 每个观点有 ≥2 个子要素 ✅ 无 Single Spotlight ✅ 关系类型匹配观点逻辑

---

### Phase 2.5：中文文字规划（⚠️ 关键步骤）

**中文是画面的主角**——图片面向中国用户，中文标题和节点标签必须醒目、占视觉重心。为每个观点决定以下文字内容：

| 文字层级 | 字数限制 | 内容 | 在画面上的位置 |
|:---|:---:|:---|:---|
| **主标题** | 4-8 字 | 观点的浓缩表达 | 画面顶部或中央偏上，最大字号，加粗 |
| **节点标签** | 2-4 字 × 2-4 个 | 每个子要素的中文关键词 | 放在对应子要素的图标旁边，中等字号 |
| **正文** | **不放正文** | 不放段落/长句 | — |

**提示词中引导中文渲染的关键词**：在提示词内直接写想要的中文字，用引号括起来，并配合以下引导短语：

```
The handwritten Chinese characters "[中文字]" appear prominently on the page, drawn in a sketchy hand-drawn marker style with visible ink texture — the Chinese text is part of the artwork itself, not a digital overlay.
```

类似地告诉模型这中文是**手绘出来的一部分**，不是文字渲染任务。

**输出格式**（在 Phase 2 视觉决策展示中增加）：

```
**中文文字**：主标题「{4-8字中文}」· 节点标签「{2-4字}」「{2-4字}」「{2-4字}」
```

---

### Phase 3：生成 1+N 段提示词

**目标**：先生成 1 段结构总览图提示词（Phase 0.5），再为每个概念（Phase 1）生成 1 段观点图提示词。共 1+N 段独立英文提示词。

#### 结构总览图提示词模板

结构总览图与观点图的本质区别：**观点图聚焦一个论点（深），结构总览图概括整篇逻辑骨架（全）**。构图不使用 Single Spotlight——必须使用展示关系和脉络的结构型构图。

```
[Opening] A close-up shot of a sketchbook page featuring a framework overview hand-drawn visual note on [{paper}] background. This is NOT a single-concept illustration — it is a structural map showing the logical flow and skeleton of the entire topic.

[Chinese Text - Primary] The page is dominated by large handwritten Chinese characters "[{Chinese_title_4-6_chars}]" drawn prominently at the top in a bold sketchy marker pen style with visible ink texture. The Chinese text is part of the hand-drawn artwork, not a digital typeset overlay. This is the framework title.

[Structure] The page maps out the complete logical structure of the topic: [{one-sentence overview of the whole logical skeleton in English}]. Composition: [{composition_type}] — [{composition_detail_description_in_2-3_sentences}].

[Node labels - Chinese] Each structural node has a small handwritten Chinese label (2-4 characters each) next to its icon. The labels are: [{node1_label}] [{node2_label}] [{node3_label}] [{node4_label}] [{node5_label}]. The labels are drawn in a smaller marker pen style, consistent with the title.

[Nodes and Connections] The diagram contains {N} nodes connected by [{connection_type}]: [{node_descriptions_in_sequence — for each node, briefly describe its icon and its role in the structure}]. Arrows or connecting lines are drawn with black ink fineliner, slightly wobbly and hand-drawn, clearly showing the flow from one node to the next. The connections between nodes are as important as the nodes themselves — this is a map of relationships, not a collection of isolated icons.

[Style] Black ink fineliner outlines, [{line_style_from_paper}], visible pen pressure, imperfect wobbly lines, [{texture_from_paper}]. The Chinese characters are rendered as hand-drawn calligraphy with marker pen strokes — slightly irregular, with ink pooling at stroke ends. Color: earthy tones with pops of [{accent_color}]. Connection arrows and lines are emphasized with colored pencil or marker shading. No drop shadows, no 3D, strictly 2D flat lay.

[Surface] [{decoration_type}], slight coffee stain in corner, folded paper edge.

[Keywords] Ballpoint pen + alcohol-based marker shading + colored pencil accents. Sketchnote, Visual Facilitation, Hand-drawn Doodling, Marker Pen Illustration, Ink and Wash Texture, Rough Edges, Imperfect Lines, Cross-Hatching Shading, Organic Typography. Handwritten Chinese calligraphy, Chinese characters as art element, Asian sketchnote style. Framework overview, logical flow diagram, structural map, process chart, hand-drawn flowchart.

[Text constraint] One Chinese title (4-6 chars) at top + {N} short Chinese node labels (2-4 chars each) next to each node. No long sentences, no paragraphs. The Chinese labels ARE the visual.

--ar 4:3 --stylize 400 --v 6.0 --no digital art, vector, cgi, smooth gradients, photorealistic, 3d render, clean lines, English text, Latin alphabet, Western typography, single icon, one concept, minimal
```

#### 结构总览图占位符说明

| 占位符 | 来源 | 说明 |
|:---|:---|:---|
| `{paper}` | Phase 0.5 色调决定 | `textured kraft paper`（温暖/日常）或 `aged white paper`（冷静/技术） |
| `{Chinese_title_4-6_chars}` | Phase 0.5 主标题 | 如「搭建专属知识库」「知识库五步法」 |
| `{one-sentence overview}` | Phase 0.5 总览主题 | 英文一句话，描述整篇的逻辑骨架 |
| `{composition_type}` | Phase 0.5 结构类型 | `Flow Pipeline` / `Stacked Layers` / `Hub & Spokes` / `Circular Flow` / `Left-to-Right Journey` |
| `{composition_detail_description}` | 构图 + 节点数 | 2-3 句英文，具体描述这个流程图的布局（如「5 nodes arranged left to right in a pipeline, connected by curved arrows, the final node highlighted with a star burst」） |
| `{node1-5_label}` | Phase 0.5 节点标签 | 2-4 字中文，如「下载工具」「AI学方法」「丢RAW」「自动Wiki」「聊天调用」 |
| `{connection_type}` | 结构类型 | `curved hand-drawn arrows` / `stacked layers with upward arrows` / `radial lines from center` / `circular looping arrows` |
| `{node_descriptions_in_sequence}` | Phase 0.5 节点 | 每个节点的英文描述（图标+角色），如「Node 1: a download icon with a small gem, representing "Download Obsidian + Agent" → Node 2: a robot reading a book, representing "AI learns the methodology"」 |
| `{line_style_from_paper}` | 纸张 | Kraft→`loose, organic, expressive lines` / Aged White→`controlled, clean, steady lines` |
| `{texture_from_paper}` | 纸张 | Kraft→`heavy paper tooth texture, visible ink bleeding at edges, slight smudging` / Aged White→`subtle paper grain, faint ink show-through` |
| `{accent_color}` | Phase 0.5 色调 | 如 `warm yellow and soft purple`、`cobalt blue and sage green` |
| `{decoration_type}` | 纸张+结构 | Kraft常用→`a few margin doodles`；Aged White常用→`a small starburst accent` |

#### 观点图提示词模板

观点图与结构总览图的区别：**结构总览图是全篇骨架（5-6个节点跨主题），观点图是单个论点的微骨架（2-4个子要素在同一个主题内）**。观点图也用结构型构图，禁止 Single Spotlight。

```
[Opening] A close-up shot of a sketchbook page featuring a micro-structure hand-drawn visual note on [{paper}] background. This illustrates the internal logic of ONE specific idea — it is a structural breakdown, not a single-icon illustration.

[Chinese Text - Primary] The page is dominated by large handwritten Chinese characters "[{Chinese_title_4-8_chars}]" drawn prominently at the top in a bold sketchy marker pen style with visible ink texture. The Chinese text is part of the hand-drawn artwork, not a digital typeset overlay. This is the idea title.

[Structure] The page breaks down one core idea: [{one-sentence concept in English}]. Composition: [{composition_type}] — [{composition_detail_description_in_2-3_sentences}]. This is a micro-structure map of a single concept's internal logic.

[Node labels - Chinese] Each element has a small handwritten Chinese label (2-4 characters each) next to its icon. The labels are: [{node1_label}] [{node2_label}] [{node3_label}] [{node4_label}]. The labels are drawn in a smaller marker pen style, consistent with the title.

[Elements and Relationships] The diagram contains {N} elements connected by [{connection_type}]: [{element_descriptions_in_sequence — for each element, briefly describe its icon and its role}]. The connections between elements clearly show [{relationship_type}]. Arrows or connecting lines are drawn with black ink fineliner, slightly wobbly and hand-drawn.

[Style] Black ink fineliner outlines, [{line_style_from_paper}], visible pen pressure, imperfect wobbly lines, [{texture_from_paper}]. The Chinese characters are rendered as hand-drawn calligraphy with marker pen strokes — slightly irregular, with ink pooling at stroke ends. Color: earthy tones with pops of [{accent_color}]. [{highlight_instruction}]. No drop shadows, no 3D, strictly 2D flat lay.

[Surface] [{decoration_type}], slight coffee stain in corner, folded paper edge.

[Keywords] Ballpoint pen + alcohol-based marker shading + colored pencil accents. Sketchnote, Visual Facilitation, Hand-drawn Doodling, Marker Pen Illustration, Ink and Wash Texture, Rough Edges, Imperfect Lines, Cross-Hatching Shading, Organic Typography. Handwritten Chinese calligraphy, Chinese characters as art element, Asian sketchnote style. [{composition_keyword}].

[Text constraint] One Chinese title (4-8 chars) at top + {N} short Chinese node labels (2-4 chars each) near their elements. No long sentences, no paragraphs. The Chinese text IS the visual.

--ar 4:3 --stylize 400 --v 6.0 --no digital art, vector, cgi, smooth gradients, photorealistic, 3d render, clean lines, English text, Latin alphabet, Western typography, single icon, one isolated concept, minimal, cover page
```

#### 观点图占位符说明

| 占位符 | 来源 | 说明 |
|:---|:---|:---|
| `{paper}` | Phase 2 纸张 | `textured kraft paper` 或 `aged white paper` |
| `{Chinese_title_4-8_chars}` | Phase 2 主标题 | 观点的 4-8 字中文浓缩 |
| `{one-sentence concept}` | Phase 1 概念 | 翻译为英文的一句话 |
| `{composition_type}` | Phase 2 构图 | `Mini Pipeline` / `Split Card` / `Arrow Flow` / `Trio Grid` / `Stacked Layers` / `Circular Flow` / `Before-After Split` |
| `{composition_detail_description}` | 构图+子要素 | 2-3 句英文，具体描述布局 |
| `{node1-4_label}` | Phase 2 子要素标签 | 2-4 字中文 |
| `{connection_type}` | 关系类型 | `curved hand-drawn arrows` / `a bold dividing line` / `stacked layers with upward arrows` / `circular looping arrows` |
| `{element_descriptions_in_sequence}` | Phase 2 子要素 | 每个要素的英文描述（图标+角色） |
| `{relationship_type}` | 关系类型 | `the step-by-step flow` / `the before-vs-after contrast` / `the cause-to-effect direction` / `the three parallel components` / `the escalating layers` / `the continuous cycle` |
| `{line_style_from_paper}` | 纸张 | Kraft→`loose, organic, expressive lines` / Aged White→`controlled, clean, steady lines` |
| `{texture_from_paper}` | 纸张 | Kraft→`heavy paper tooth texture, visible ink bleeding at edges, slight smudging` / Aged White→`subtle paper grain, faint ink show-through` |
| `{accent_color}` | Phase 2 点缀色 | 如 `sage green`、`cobalt blue`、`coral red` |
| `{highlight_instruction}` | 关系类型 | Steps/Pipeline→`Connection arrows are emphasized with colored pencil shading` / Contrast→`The dividing line is highlighted with yellow marker` / Hierarchy→`Each layer has a subtle colored pencil tint` / 默认→`Red circles highlight key Chinese characters. Yellow highlighter marks under the main title` |
| `{decoration_type}` | 纸张 | Kraft常用→`a few margin doodles`；Aged White常用→`a small starburst accent` |
| `{composition_keyword}` | 构图 | `Mini Pipeline`→`hand-drawn flowchart, step diagram` / `Split Card`→`comparison diagram, before-after sketch` / `Circular Flow`→`cycle diagram, loop illustration` / `Trio Grid`→`component breakdown, parts diagram`

---

### Phase 4：最终输出

**输出格式**（对话界面展示）：

```
## 🗺 全篇结构总览
{结构总览主题 + 结构类型 + 节点列表}

---

## 📖 核心概念提取
{概念表格 — 如果 Phase 1 已经展示过了，这里简写"已确认 {N} 个概念"即可}

---

### 🗺 结构总览图

**视觉方案**：{纸张} · {点缀色} · {构图} · 中文「{主标题}」· {N}个节点

**提示词**：

\`\`\`
{结构总览图提示词}
\`\`\`

---

{对每个观点图逐一输出：}

### 🖼 结构观点 {序号}：{一句话}

**微骨架**：{关系类型} · {构图} · 中文「{主标题}」· 节点「{标签1}」「{标签2}」「{标签3}」

**提示词**：

\`\`\`
{Phase 3 填充后的完整英文提示词}
\`\`\`

---

{所有概念输出完毕后：}

## 📋 使用说明

1. 将以上 1+N 段提示词分别粘贴到 **Midjourney / Stable Diffusion / DALL-E / 通义万相** 中
2. 第一段是结构总览图——展示了整篇内容的逻辑骨架和脉络；后面 {N} 段是结构观点图——每张深挖一个论点的内部拆解（子要素+关系），信息密度高，有收藏价值
3. 如果某张图的中文字渲染不理想（乱码/缺笔），可尝试：重新生成同一段提示词 → 或者用 PS / Canva / 黄油相机 覆盖贴中文手写体（推荐字体：黄油相机手写体 / 站酷快乐体 / 方正静蕾体 / 沐瑶软笔手写体）
4. 如果某张图的风格或构图不理想，告诉我编号，我调整提示词重出

如需对某个概念调整视觉方向，直接告诉我编号和需求。
```

**检查点**：✅ 1+N 段提示词全部输出 ✅ 结构总览图为第一段 ✅ 观点图紧随其后

---

## 完成后

本 skill 为纯文本生成，无写操作。无需写入 Obsidian Vault 或更新索引。
