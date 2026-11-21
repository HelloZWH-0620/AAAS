# AAAS 题库文件编写指南

**摘要** 本文规定 AAAS v1 题库交换文件的组成、字段取值与内容表达方式，并给出从空文件到可发布试卷的编写流程。文件采用 UTF-8 编码的 YAML，由版本号、元信息、渲染约定、题型注册表和大题列表构成；大题包含材料层，小题包含内容层与评分层。文中列出机器可执行的校验项与人工自检项，并附字段表、题型表、表达式对照表及最小文件骨架。

**关键词** 题库交换格式；YAML；Markdown；LaTeX；JSON Schema

---

## 1 引言

AAAS（题库交换格式，版本 1）用于在出题、组卷与作答软件之间传递整份试卷。一个文件对应一份试卷，包含题型注册表、材料与题干、答案、采分点和来源标注。

本指南面向需要独立编写 AAAS 文件的人员。第 3 节给出文件骨架，第 4 节定义各层字段，第 5 节说明内容表达，第 6 至 7 节规定题型与来源，第 8 节给出编写步骤，第 9 节给出校验方法。按上述顺序操作即可写出一份可被软件读取的文件。

本指南规定文件格式，不规定渲染器的视觉实现。文中「须」表示规范性要求，「宜」表示推荐做法。配套文件：`example.yaml`（完整示例）、`aaas.schema.json`（机器校验）、`README.md`（项目概览）。

## 2 术语与记号

**表 1　类型与记号**

| 记号 | 含义 |
| --- | --- |
| `int` | 整数 |
| `float` | 浮点数 |
| `string` | 字符串 |
| `bool` | 布尔值，取 `true` 或 `false` |
| `enum` | 枚举，只能取表内列出的值 |
| `map` | 映射，由若干具名字段构成 |
| `list` / `string[]` | 列表 / 字符串列表 |
| `{{n}}` | 题干中的第 n 个空位，n 从 1 起 |
| `_ref` | 外链字段后缀，值为外部文件路径 |
| 条件必填 | 仅在特定条件下必须出现，见对应字段说明 |

**表 2　通用约束**

| 对象 | 约束 |
| --- | --- |
| 版本号 | `version` 恒为 `1` |
| 题号 | 大题题号在本层唯一，小题题号在所属大题内唯一；均从 1 起 |
| 分值 | 所有 `points` 与 `key_points[].score` 均为 0.5 的整数倍 |
| 计数 | 小题数、选项数、空位数由列表或标记的长度推导，不设计数字段 |
| 日期 | `YYYY-MM-DD`，以字符串形式书写，须加引号 |
| 编码 | UTF-8，无 BOM |

## 3 文件组成

```
AAAS 文件（一个 YAML 文档）
├── version                    版本号
├── meta                       题库元信息（可选）
├── content                    填充与渲染约定（可选）
├── category[]                 题型注册表，先于引用声明
└── questions[]                大题列表
    ├── number / points        题号 / 分值
    ├── ask / material         引导语 / 材料（材料层）
    ├── tags / origin          知识点 / 来源
    └── items[]                小题列表
        ├── stem / options     题干 / 选项（内容层）
        ├── answer             答案
        └── points / key_points / max_words    评分层
```

`category` 是注册表，`questions[].items[].category` 引用其中的 `key`。引用须命中，未登记的题型不得使用。材料层由同一大题下各小题共用；内容层和评分层属于单个小题。

## 4 字段定义

### 4.1 根

**表 3　根字段**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `version` | int | 是 | 固定 `1` |
| `meta` | map | 否 | 题库元信息，见表 4 |
| `content` | map | 否 | 渲染约定，见表 5 |
| `category` | list | 是 | 题型注册表，至少一项，见表 6 |
| `questions` | list | 是 | 大题列表，至少一项，见表 7 |

### 4.2 元信息（meta）

**表 4　`meta` 字段**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `title` | string | 否 | 题库名称 |
| `subject` | string | 否 | 学科名；与 CSES 的 `subjects[].name` 取值一致，合卷写「理综」「文综」 |
| `grade` | string | 否 | 年级，写中文，如「高一」「七年级」「高三」 |
| `authors` | string[] | 否 | 作者列表，按署名顺序排列 |
| `updated` | string | 否 | 最后修改日期，格式 `YYYY-MM-DD`，须加引号 |

### 4.3 渲染约定（content）

**表 5　`content` 字段**

| 字段 | 类型 | 必填 | 默认值 | 说明 |
| --- | --- | --- | --- | --- |
| `default_source` | enum | 否 | `inline` | `inline` / `file`；编辑器新建题目时的表单初值，不参与文件校验 |
| `math` | enum | 否 | `auto` | `latex` 强制解析 `$...$`；`auto` 成对才解析；`none` 按字面量 |
| `chemistry` | enum | 否 | `none` | `mhchem` 时解析 `\ce{}`；`none` 时不解析 |
| `answer_input` | enum | 否 | `text` | `text` / `latex`；作答侧是否接受 LaTeX 输入 |
| `allow_html` | bool 或 enum | 否 | `tables` | 裸 `false` 不放行；`tables` 仅放行表格标签；`all` 放行白名单内标签 |
| `asset_dir` | string | 否 | `assets` | 图片与外链文件的基准目录，相对本文件所在位置 |

### 4.4 题型注册表（category）

**表 6　`category[]` 字段**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `key` | string | 是 | 题型标识，全库唯一；宜用小写字母、数字与短横线 |
| `name` | string | 是 | 显示名 |
| `auto_grade` | bool | 是 | 是否可机器判分 |
| `fields` | string[] | 是 | 该题型在小题层必须出现的字段名 |
| `answer_type` | string | 是 | `answer` 的取值形态说明，供校验器读取 |

`fields` 与 `answer_type` 是给程序使用的元数据。新增题型须在本表登记，再在小题中引用其 `key`。

### 4.5 大题（questions）

**表 7　`questions[]` 字段**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `number` | int | 是 | 大题题号，≥1，本层唯一 |
| `points` | float | 是 | 大题分值，等于各小题 `points` 之和 |
| `ask` | string | 否 | 引导语，行内 Markdown；与 `ask_ref` 二选一 |
| `ask_ref` | string | 否 | 引导语外链；与 `ask` 二选一 |
| `material` | string | 否 | 材料，块级 Markdown；与 `material_ref` 二选一 |
| `material_ref` | string | 否 | 材料外链；与 `material` 二选一 |
| `tags` | string[] | 否 | 知识点标签，与小题 `tags` 取并集 |
| `origin` | map | 否 | 来源，见表 9 |
| `items` | list | 是 | 小题列表，至少一项，见表 8 |

无材料时 `ask` 与 `material` 写空串，保留大题层。

### 4.6 小题（items）

**表 8　`items[]` 字段**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `number` | int | 是 | 小题题号，≥1，本大题内唯一 |
| `category` | string | 是 | 命中 `category[].key` |
| `points` | float | 是 | 小题分值，0.5 的整数倍 |
| `stem` | string | 条件必填 | 题干；与 `stem_ref` 二选一，必居其一 |
| `stem_ref` | string | 条件必填 | 题干外链；与 `stem` 二选一，必居其一 |
| `answer` | 依题型 | 条件必填 | 见表 13；与 `answer_ref` 二选一，必居其一 |
| `answer_ref` | string | 条件必填 | 答案外链；与 `answer` 二选一 |
| `options` | string[] | `choose` 必填 | 2 至 8 项，顺序对应 A、B、C…；仅行内 Markdown |
| `key_points` | list | 否 | 采分点列表，每项为 `{point, score}`，见 6.3 节 |
| `max_words` | int | 否 | 作答字数上限 |
| `explanation` | string | 否 | 解析；与 `explanation_ref` 二选一 |
| `explanation_ref` | string | 否 | 解析外链；与 `explanation` 二选一 |
| `tags` | string[] | 否 | 省略时继承大题 `tags` |
| `origin` | map | 否 | 省略时继承大题 `origin` |

### 4.7 来源（origin）

**表 9　`origin` 字段**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `type` | enum | 是 | `human` / `ai` / `adapted` / `excerpt` |
| `source` | string | 否 | 出处自由文本，用于展示 |
| `ref` | map | 否 | `{number, url}`；`number` 为原题题号，无编号写 0；`url` 可为空串 |
| `model` | string | `ai` 必填 | 生成模型标识 |
| `generated_at` | string | `ai` 必填 | 生成日期，`YYYY-MM-DD` |
| `reviewed` | bool | `ai` 必填 | 是否经人工审校；`false` 的题不进入正式练习 |
| `reviewed_by` | string | 否 | 审校人；`reviewed` 为 `true` 时应填写 |

## 5 内容表达

### 5.1 槽位来源

可设置外链的字段共五个：`ask`、`material`、`stem`、`answer`、`explanation`。`options` 与 `key_points` 是结构字段，不设外链。

**表 10　三种内容来源**

| 来源 | 字段写法 | 适用内容 | 解析方式 |
| --- | --- | --- | --- |
| 内联 | `stem: '...'` | 短题干、选项、答案、采分点 | 按 `content` 配置解析 |
| Markdown 文件 | `stem_ref: "items/q1.md"` | 长材料、图文混排、表格 | Markdown，可含 `$...$` |
| LaTeX 文件 | `stem_ref: "items/q1.tex"` | 公式推导、符号密集的题干 | 整段 LaTeX，不解析 Markdown |

规则：

1. 每个槽位独立选择来源。同一小题可同时出现外链材料与内联题干。
2. 本体与 `_ref` 不得同时出现；两者都不写时该槽位视为缺失。
3. 外链路径相对 `content.asset_dir`。
4. `.tex` 不解析 Markdown，不宜承载大段中文；中文正文用 `.md`。
5. 编辑已有题目时，外链槽位修改的是外部文件；改为内联须读入文件内容并删除 `_ref`，反向操作称为外链化。

### 5.2 文本、公式与化学式

公式以 LaTeX 源码存入字段，排版参数由渲染器在读取时确定；字段中不保存渲染后的图片。

- 行内公式用 `$...$`，独占一段的公式用 `$$...$$`，多行对齐用 `\begin{aligned}`。
- 单位用正体并与数值以 `\,` 分隔：`$10\,\mathrm{m/s^2}$`。
- 化学方程式用 `\ce{}`：`\ce{2SO2(g) + O2(g) <=> 2SO3(g)}`，其中 `<=>` 表示可逆箭头。结构式、键线式和实验装置图使用图片。
- 生物基因型写作 `$A\_B\_$`；杂交符号写作 `\times`。
- 公式编号写入正文文本，不使用 `\label` 与 `\ref`。禁用 `\usepackage`、`\newcommand`、`\includegraphics` 与 TikZ。
- YAML 双引号内的 `\,` 属非法转义。含 LaTeX 的字符串使用单引号或 `|-` 块标量。
- `content.math` 取 `latex` 时，落单的 `$` 会被判为错误；取 `auto` 时仅成对的 `$` 被解析。综合卷宜用 `auto`，语文等 `$5` 表示货币的卷子宜用 `none`。

**表 11　学科配置建议**

| 学科 | `math` | `chemistry` | 编写要点 |
| --- | --- | --- | --- |
| 数学 | `latex` | `none` | 多行用 `aligned`；几何图用 SVG |
| 物理 | `auto` | `none` | 单位正体加 `\,`；矢量用 `\vec{}`；受力图用 SVG |
| 化学 | `auto` | `mhchem` | 方程式用 `\ce{}`；结构式与装置图用图片；需限定有效数字时答案写字符串 |
| 生物 | `auto` | `none` | 基因型用斜体；系谱图与遗传图解用 SVG |
| 语文 / 英语 / 政史地 | `none` | `none` | `$` 按字面量处理 |

### 5.3 表格

表格是题干或材料的内容，不设独立题型。表格以 GFM 语法直接写入 `material` 或 `stem`：

```yaml
stem: |-
  测得数据如下：

  | 浓度 / ($\mathrm{mol\cdot L^{-1}}$) | $\ce{SO2}$ | $\ce{O2}$ | $\ce{SO3}$ |
  | --- | --- | --- | --- |
  | 起始 | $1.0$ | $0.5$ | $0$ |
  | 转化 | $0.8$ | {{1}} | $0.8$ |
```

1. 单元格内可写 `{{n}}`，该格渲染为输入框；题型仍为 `blank`，答案统一列入 `answer`。
2. 空位按先行后列、先左后右编号，跨行继续递增。
3. 各行列数须相同，表头列数与数据行一致。
4. 一格宜只放一个空位，空位不跨单元格。
5. `{{n}}` 只出现在 `stem` 或 `stem_ref` 指向的内容中。`material` 由多个小题共用，其中的空位无法归属，故不得写空位。
6. GFM 不支持合并单元格；需要时使用 `allow_html: tables` 放行 HTML 表格，或将表格转为图片。

### 5.4 图片

1. 优先使用 SVG；照片用 WebP；位图用 PNG 且分辨率不低于 300 DPI。
2. 路径相对 `content.asset_dir`。
3. alt 文本填写「图号 + 图名」，正文以「如图 1 所示」指代，不依赖自动编号。
4. 涉及国界或行政区划的地图须使用带审图号的底图，不得自行绘制国界线。

### 5.5 空位

1. 空位写作 `{{n}}`，n 从 1 连续递增，不重复。
2. 空位个数等于 `answer` 列表长度，编号与答案顺序一一对应。
3. 空位只出现在题干，不出现在材料、引导语、解析与答案中。
4. 文本答案多种写法以 `|` 分隔；写 `""` 表示交专家批改。

## 6 题型与判分

### 6.1 推荐注册项

`category` 为注册表，以下四项为推荐配置。新增题型须登记 `key`、`fields` 与 `answer_type`。

**表 12　推荐题型**

| `key` | `name` | `auto_grade` | `fields` | 判分方式 |
| --- | --- | --- | --- | --- |
| `choose` | 选择题 | `true` | `[options, answer]` | 按选项序号比对 |
| `blank` | 填空题 | `true` | `[answer]` | 文本归一化或数值容差 |
| `short` | 简答题 | `false` | `[answer]` | 采分点或专家评阅 |
| `judge` | 判断题 | `true` | `[answer]` | 布尔值比对 |

### 6.2 答案形态

**表 13　各题型 `answer` 取值**

| 题型 | 类型 | 示例 |
| --- | --- | --- |
| `choose` | int 或 int[] | `2`；`[1, 3]`（升序） |
| `blank` | string 或 map[] | `"10"`；`{value: 10, tol: 0.02, unit: N}` |
| `short` | string | 分点书写的参考答案，不使用「略」 |
| `judge` | bool | `true` / `false` |

判断题的答案不得写作 `"true"`、`"对"`、`T`、`√` 等形式。选择题序号从 1 起，须在 `1..len(options)` 范围内且不重复。

### 6.3 采分点与容差

1. `key_points` 每项含 `point`（一句话描述）与 `score`（0.5 的整数倍）。
2. 各项 `score` 之和等于该小题 `points`。缺省 `key_points` 时按整体评分。
3. 数值答案对象含 `value`（标准答案）、`tol`（相对容差，如 `0.02` 表示 ±2%）与 `unit`（单位）。
4. 需要限定有效数字时改用字符串精确匹配，容差判分不区分 `0.40` 与 `0.4`。

## 7 来源标注

`origin` 可写在题库下的任一题层，省略视为人工原创。小题未写时继承大题的取值。

**表 14　`origin.type` 取值**

| `type` | 含义 | 附加要求 |
| --- | --- | --- |
| `human` | 人工原创 | 无 |
| `ai` | 模型生成 | 须写全 `model`、`generated_at`、`reviewed` |
| `adapted` | 改编自原题 | 宜填写 `source` 与 `ref` |
| `excerpt` | 引用原题 | 宜填写 `source` 与 `ref` |

来源信息使用结构化字段而非标签，以便按来源过滤，并将 `ai`、`adapted`、`excerpt` 渲染为对应角标。

## 8 编写流程

1. 新建 UTF-8 编码的 `xxx.yaml`，写入 `version: 1`。
2. 填写 `meta`，其中 `subject` 与 CSES 的学科名保持一致。
3. 填写 `content`，按学科设定 `math`、`chemistry` 与 `answer_input`。
4. 在 `category` 登记题型；常规试卷可直接采用表 12 的四项。
5. 逐题建立 `questions[]` 条目，填写 `number`、`points`，以及 `ask`、`material`。
6. 逐条建立 `items[]`，填写 `number`、`category`、`points`、`stem`、`answer`；选择题补 `options`。
7. 按 5.1 确定每个槽位的来源，需要外链时改用对应的 `_ref` 字段。
8. 填空题核对 `{{n}}` 编号与 `answer` 顺序；简答题填写 `key_points`；理科填空题设定 `tol` 与 `unit`。
9. 需要标注来源时填写 `origin`，AI 生成题补齐三个必填字段。
10. 按第 9 节执行校验，确认通过后发布。

附录 A 给出可直接复制的最小骨架。

## 9 校验

### 9.1 机器校验

`aaas.schema.json` 依据 JSON Schema draft-07 编写，可检查：必填字段、枚举取值、`version` 取值、`_ref` 与本体互斥、题干与答案是否存在、选项数量、分值是否为 0.5 的倍数、AI 题的三个必填字段、数值答案是否含 `value`。

编辑器配置：在文件首行加入

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/HelloZWH-0620/AAAS/main/aaas.schema.json
```

或在编辑器的 YAML 设置中把 `*.yaml` 映射到该 Schema 地址。命令行校验：

```bash
pip install check-jsonschema
check-jsonschema --schemafile aaas.schema.json example.yaml
```

`answer` 的具体形态由注册表 `answer_type` 决定，而注册表属于数据，Schema 无法把题型与答案形态绑定。Schema 只校验答案是否为合法形态之一；题型对应的形态校验列入表 15。

### 9.2 人工与程序自检项

**表 15　发布前检查项**

| 编号 | 检查内容 |
| --- | --- |
| C1 | `version` 为 `1`；`category`、`questions` 存在 |
| C2 | 小题 `category` 命中注册表；`fields` 所列字段齐全 |
| C3 | 大题 `points` 等于各小题 `points` 之和 |
| C4 | `key_points[].score` 之和等于该小题 `points` |
| C5 | `{{n}}` 从 1 连续、无重复，个数等于 `answer` 长度 |
| C6 | `{{n}}` 不出现在材料、引导语、解析与答案中 |
| C7 | `choose` 的 `answer` 在 `1..len(options)` 内且不重复；`judge` 的 `answer` 为布尔 |
| C8 | 本体与 `_ref` 不同时出现；`options`、`key_points` 无 `_ref` |
| C9 | 表格各行列数一致；含 `{{n}}` 的表格位于 `stem` |
| C10 | 外链与图片路径存在、不越出 `asset_dir`、文件为 UTF-8 |

C1 至 C9 属于发布阻断项。C10 在草稿态记为警告，发布态记为错误。

## 10 作答数据

作答数据与题库共用三个内容来源：直接输入、上传 `.md`、上传 `.tex`。作答内容写入题库既有的槽位，由题号与空位编号定位，不替换题库中的题干或材料。

| 约束 | 内容 |
| --- | --- |
| 槽位固定 | 仅填写 `questions[].number`、`items[].number` 与 `{{n}}` 指定的位置 |
| 格式限制 | `content.answer_input` 为 `text` 时拒收 `.tex`；为 `latex` 时三种来源均可 |
| 判分方式 | `choose`、`judge` 比对答案；`blank` 按文本归一化或数值容差；`short` 按采分点或专家评阅 |

## 附录 A　最小文件骨架

```yaml
version: 1
meta:
  title: "示例试卷"
  subject: 物理
  grade: 高三
  authors: [示例作者]
  updated: "2026-11-21"
content:
  math: auto
  chemistry: none
  answer_input: text
category:
  - key: judge
    name: 判断题
    auto_grade: true
    fields: [answer]
    answer_type: bool
questions:
  - number: 1
    points: 2.0
    ask: ""
    material: ""
    items:
      - number: 1
        category: judge
        points: 2.0
        stem: '滑动摩擦力的大小一定等于 $\mu mg$。'
        answer: false
```

完整示例见 `example.yaml`。

## 参考文献

[1] SmartTeachCN. CSES: The Course Schedule Exchange Schema[EB/OL]. https://github.com/SmartTeachCN/CSES.

[2] YAML Language Development Team. YAML 1.2.2 Specification[EB/OL]. https://yaml.org/spec/1.2.2/.

[3] JSON Schema Team. JSON Schema Draft 07[EB/OL]. https://json-schema.org/specification-links.html#draft-7.

[4] CommonMark Team. CommonMark Spec[EB/OL]. https://spec.commonmark.org/.

[5] Kahn Academy. KaTeX Documentation[EB/OL]. https://katex.org/docs/.

[6] mhchem Project. mhchem Documentation[EB/OL]. https://mhchem.github.io/MathJax-mhchem/.
