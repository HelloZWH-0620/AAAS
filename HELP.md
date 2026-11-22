# AAAS 题库文件编写指南

**摘要** 本文规定 AAAS v1 题库交换文件的组成、字段取值与内容表达方式，并给出从空文件到可发布题目的编写流程。文件采用 UTF-8 编码的 YAML，由版本号、可选的元信息与渲染约定、题目列表构成；每道题自包含，可独立抽出并保留卷面题号，也可选用题组包装共享材料。文中给出四种标准题型的答案文法与性质、字段表、机器校验项与人工自检项，并附最小文件骨架。

**关键词** 题库交换格式；YAML；Markdown；LaTeX；JSON Schema

---

## 1 引言

AAAS（题库交换格式，版本 1）用于在出题、组卷与作答软件之间传递题目。一个文件可含一道或多道题，每道题自包含题型、题干、答案、采分点与来源标注，抽出后不依赖其他题目即可使用。

本指南面向需要独立编写 AAAS 文件的人员。第 3 节给出文件骨架，第 4 节定义各层字段，第 5 节说明内容表达，第 6 至 7 节规定题型、答案文法与来源，第 8 节给出编写步骤，第 9 节给出校验方法。配套文件：`example.yaml`（完整示例）、`aaas.schema.json`（机器校验）、`README.md`（项目概览）。

本指南规定文件格式，不规定渲染器的视觉实现。文中「须」表示规范性要求，「宜」表示推荐做法。

## 2 术语与记号

**表 1　类型与记号**

| 记号 | 含义 |
| --- | --- |
| `int` | 整数值，用于题号与选项序号 |
| `number` | 数值，用于分值、容差与标准答案；允许整数写法 |
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
| 题号 | `number` 可选；省略时按 `questions` 数组顺序推导为索引 + 1；写出时保留原值，供抽出后沿用 |
| 题型 | 固定为 `choose` / `blank` / `short` / `judge` 四种，不设自定义题型 |
| 分值 | 所有 `points` 与 `key_points[].score` 均为 0.5 的整数倍 |
| 计数 | 选项数、空位数由内容长度推导，不设计数字段 |
| 日期 | `YYYY-MM-DD`，以字符串形式书写，须加引号 |
| 编码 | UTF-8，无 BOM |

## 3 文件组成

```
AAAS 文件（一个 YAML 文档）
├── version                    版本号
├── meta                       题库元信息（可选）
├── content                    填充与渲染约定（可选）
└── questions[]                题目列表，至少一项
    ├── number / points        卷面题号 / 分值（均可选）
    ├── ask / material         引导语 / 材料（共享层）
    ├── tags / origin          知识点 / 来源（共享层）
    ├── 单题形态：type / stem / answer / options / key_points
    └── 题组形态：members[]
        └── 成员：type / points / stem / answer / options / key_points
```

题目有两种互斥形态：

1. **单题**：题目层直接写 `type`、`stem`、`answer`，其余字段按题型补充。
2. **题组**：题目层写 `members`，由若干成员组成，材料与来源写在题目层共享。出现 `members` 后，题目层不得再写 `type`、`stem`、`answer` 等单题字段。

题组成员不写题号，序号由 `members` 数组顺序推导。成员题号仅在整卷语境中有意义，由组卷软件按题目层 `number` 与成员位置合成。

## 4 字段定义

### 4.1 根

**表 3　根字段**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `version` | int | 是 | 固定 `1` |
| `meta` | map | 否 | 题库元信息，见表 4 |
| `content` | map | 否 | 渲染约定，见表 5 |
| `questions` | list | 是 | 题目列表，至少一项，见表 7 |

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

### 4.4 题型（type）

**表 6　`type` 取值**

| `type` | 名称 | 必填字段 | 答案类型 | 判分方式 |
| --- | --- | --- | --- | --- |
| `choose` | 选择题 | `options`、`answer` | `index` 或 `index_list` | 按选项序号比对 |
| `blank` | 填空题 | `answer` | `text` 或 `blank_list` | 文本归一化或数值容差 |
| `short` | 简答题 | `answer` | `text` | 采分点或专家评阅 |
| `judge` | 判断题 | `answer` | `bool` | 布尔值比对 |

题型为封闭枚举。第 6 节给出各答案类型的文法与性质。题目层与题组成员层均使用本表的 `type`，字段名相同。

### 4.5 题目（questions）

**表 7　`questions[]` 字段**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `number` | int | 否 | 卷面题号，≥1；抽出试卷时保留，省略时按数组顺序推导 |
| `points` | number | 否 | 分值，0.5 的整数倍；题组模式下等于各成员 `points` 之和 |
| `type` | enum | 单题必填 | 见表 6；题组模式下不得出现 |
| `ask` | string | 否 | 引导语，行内 Markdown；与 `ask_ref` 二选一 |
| `ask_ref` | string | 否 | 引导语外链；与 `ask` 二选一 |
| `material` | string | 否 | 材料，块级 Markdown；与 `material_ref` 二选一 |
| `material_ref` | string | 否 | 材料外链；与 `material` 二选一 |
| `stem` / `stem_ref` | string | 单题条件必填 | 题干，二者必居其一，不得并存 |
| `answer` / `answer_ref` | 依题型 / string | 单题条件必填 | 答案，二者必居其一，不得并存 |
| `options` | string[] | `choose` 必填 | 2 至 8 项，顺序对应 A、B、C…；仅行内 Markdown |
| `key_points` | list | 否 | 采分点列表，每项为 `{point, score}`，见 6.3 节 |
| `max_words` | int | 否 | 作答字数上限 |
| `explanation` / `explanation_ref` | string | 否 | 解析，二者不得并存 |
| `tags` | string[] | 否 | 知识点标签，题组成员省略时继承本题取值 |
| `origin` | map | 否 | 来源，见表 9 |
| `members` | list | 题组必填 | 题组成员列表，至少一项，见表 8 |

`members` 与 `type` / `stem` / `stem_ref` / `answer` / `answer_ref` / `options` / `key_points` / `explanation` / `explanation_ref` 互斥。无材料时 `ask` 与 `material` 可省略。

### 4.6 题组成员（members）

**表 8　`members[]` 字段**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `type` | enum | 是 | 见表 6 |
| `points` | number | 否 | 分值，0.5 的整数倍 |
| `stem` / `stem_ref` | string | 条件必填 | 题干，二者必居其一，不得并存 |
| `answer` / `answer_ref` | 依题型 / string | 条件必填 | 答案，二者必居其一，不得并存 |
| `options` | string[] | `choose` 必填 | 2 至 8 项 |
| `key_points` | list | 否 | 采分点列表 |
| `max_words` | int | 否 | 作答字数上限 |
| `explanation` / `explanation_ref` | string | 否 | 解析，二者不得并存 |
| `tags` | string[] | 否 | 省略时继承题目层 `tags` |
| `origin` | map | 否 | 省略时继承题目层 `origin` |

成员不设 `number`。材料写在题目层，由全部成员共享；`{{n}}` 空位不得写入材料。

### 4.7 来源（origin）

**表 9　`origin` 字段**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `kind` | enum | 是 | `human` / `ai` / `adapted` / `excerpt` |
| `description` | string | 否 | 出处描述，自由文本，含原题题号时写入此处 |
| `url` | string | 否 | 原题链接，没有可写空串 |
| `model` | string | `ai` 必填 | 生成模型标识 |
| `generated_at` | string | `ai` 必填 | 生成日期，`YYYY-MM-DD` |
| `reviewed` | bool | `ai` 必填 | 是否经人工审校；`false` 的题不进入正式练习 |
| `reviewed_by` | string | 否 | 审校人；`reviewed` 为 `true` 时应填写 |

`kind` 省略整块 `origin` 时视为 `human`。`adapted` 与 `excerpt` 宜填写 `description` 与 `url`。

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

1. 每个槽位独立选择来源。同一题可同时出现外链材料与内联题干。
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
5. `{{n}}` 只出现在 `stem` 或 `stem_ref` 指向的内容中。`material` 由题组各成员共用，其中的空位无法归属，故不得写空位。
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

### 6.1 答案文法

答案形态用下列产生式定义。`Number` 为 JSON 数值，`String` 为 Unicode 字符串，`Int` 为整数值。

```
Answer     ::= Boolean | Index | IndexList | Text | Numeric | BlankList
Boolean    ::= "true" | "false"
Index      ::= n ∈ Int, 1 ≤ n ≤ |options|
IndexList  ::= "[" Index ("," Index)* "]"      且严格升序、无重复
Text       ::= String                           多种可接受写法以 "|" 分隔
Numeric    ::= {"value": Number, ["tol": Number≥0], ["unit": String]}
BlankList  ::= "[" (Text | Numeric)+ "]"        长度 = stem 中 {{n}} 的个数
```

`choose` 的 `Index` 与 `IndexList` 受 `options` 长度约束；`uniqueItems` 禁止重复序号。`Numeric.tol` 为相对容差，`0.02` 表示 ±2%，省略表示精确相等。

### 6.2 答案类型性质

**表 12　答案类型性质**

| 类型 | 载体 | 有序 | 唯一 | 绑定题型 | 判分算法 |
| --- | --- | --- | --- | --- | --- |
| `bool` | 标量 | — | — | `judge` | 字面相等 |
| `index` | 标量 | — | — | `choose` 单选 | 数值相等 |
| `index_list` | 数组 | 是 | 是 | `choose` 多选 | 集合相等 |
| `text` | 标量 | — | — | `short`、`blank` 单空 | 归一化后匹配 |
| `numeric` | 对象 | — | — | `blank` 数值空 | 绝对误差 ≤ `tol × value` |
| `blank_list` | 数组 | 是 | 否 | `blank` 多空 | 按 `{{n}}` 逐空判分 |

`index` 与 `index_list` 为离散索引，须写整数值。JSON Schema draft-07 的 `integer` 允许 `1.0` 这类零小数写法，编写时仍应写 `1`。数值容差不区分有效数字；需要限定 `0.40` 与 `0.4` 时改用 `text` 精确匹配。

判断题答案不得写作 `"true"`、`"对"`、`T`、`√` 等形式。上述绑定由 `aaas.schema.json` 内联的 `if/then` 执行，形态错配在编辑器中即可报错。

### 6.3 采分点与容差

1. `key_points` 每项含 `point`（一句话描述）与 `score`（0.5 的整数倍）。
2. 各项 `score` 之和等于该小题 `points`。缺省 `key_points` 时按整体评分。
3. 数值答案对象含 `value`（标准答案）、`tol`（相对容差）与 `unit`（单位）。
4. 需要限定有效数字时改用字符串精确匹配，容差判分不区分 `0.40` 与 `0.4`。

## 7 来源标注

`origin` 可写在题目层或题组成员层，省略视为 `human`。成员未写时继承题目层取值。

`kind` 的四种取值见表 9。`ai` 须写全 `model`、`generated_at`、`reviewed`；`reviewed: false` 的题不进入正式练习。来源信息使用结构化字段而非标签，以便按来源过滤，并将 `ai`、`adapted`、`excerpt` 渲染为对应角标。

## 8 编写流程

1. 新建 UTF-8 编码的 `xxx.yaml`，写入 `version: 1`。
2. 填写 `meta`，其中 `subject` 与 CSES 的学科名保持一致。
3. 填写 `content`，按学科设定 `math`、`chemistry` 与 `answer_input`。
4. 确定题目形态：单题直接写 `type`；多题共享材料时写 `members`。
5. 逐题建立 `questions[]` 条目，按需写 `number`、`points`、`ask`、`material`。
6. 填写 `type`、`stem`、`answer`；选择题补 `options`，简答题填 `key_points`。
7. 按 5.1 确定每个槽位的来源，需要外链时改用对应的 `_ref` 字段。
8. 填空题核对 `{{n}}` 编号与 `answer` 顺序；理科填空题设定 `tol` 与 `unit`。
9. 需要标注来源时填写 `origin`，AI 生成题补齐三个必填字段。
10. 按第 9 节执行校验，确认通过后发布。

附录 A 给出可直接复制的最小骨架。

## 9 校验

### 9.1 机器校验

`aaas.schema.json` 依据 JSON Schema draft-07 编写，可检查：必填字段、枚举取值、`version` 取值、`_ref` 与本体互斥、题干与答案是否存在、单题与题组形态互斥、选项数量、分值是否为 0.5 的倍数、题型与答案形态是否匹配、AI 题的三个必填字段、数值答案是否含 `value`。

编辑器配置：在文件首行加入

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/HelloZWH-0620/AAAS/main/aaas.schema.json
```

或在编辑器的 YAML 设置中把 `*.yaml` 映射到该 Schema 地址。命令行校验：

```bash
pip install check-jsonschema
check-jsonschema --schemafile aaas.schema.json example.yaml
```

JSON Schema 不检查跨元素的取值关系。题号唯一性、分值求和、空位个数、选项序号范围等依赖题目内容，列入表 13。

### 9.2 人工与程序自检项

**表 13　发布前检查项**

| 编号 | 检查内容 | 执行方 |
| --- | --- | --- |
| C1 | `version` 为 `1`；`questions` 存在且非空 | Schema |
| C2 | `type` 取值合法；`choose` 有 `options`，其余题型无多余字段 | Schema |
| C3 | 本体与 `_ref` 不同时出现；`options`、`key_points` 无 `_ref` | Schema |
| C4 | 题型与答案形态匹配；`judge` 的 `answer` 为布尔 | Schema |
| C5 | 分值为 0.5 的整数倍；AI 题三个必填字段齐全 | Schema |
| C6 | `number` 在同一层级内不重复 | 校验器 |
| C7 | 题组 `points` 等于各成员 `points` 之和 | 校验器 |
| C8 | `key_points[].score` 之和等于所属题 `points` | 校验器 |
| C9 | `{{n}}` 从 1 连续、无重复，个数等于 `answer` 长度 | 校验器 |
| C10 | `{{n}}` 不出现在材料、引导语、解析与答案中 | 校验器 |
| C11 | `choose` 的 `answer` 在 `1..len(options)` 内 | 校验器 |
| C12 | 表格各行列数一致；含 `{{n}}` 的表格位于 `stem` | 校验器 |
| C13 | 外链与图片路径存在、不越出 `asset_dir`、文件为 UTF-8 | 校验器 |

C1 至 C12 属于发布阻断项。C13 在草稿态记为警告，发布态记为错误。

## 10 作答数据

作答数据与题库共用三个内容来源：直接输入、上传 `.md`、上传 `.tex`。作答内容写入题库既有的槽位，由题目 `number`、成员序号与 `{{n}}` 定位，不替换题库中的题干或材料。

| 约束 | 内容 |
| --- | --- |
| 槽位固定 | 仅填写题目 `number`（或推导序号）、成员序号与 `{{n}}` 指定的位置 |
| 格式限制 | `content.answer_input` 为 `text` 时拒收 `.tex`；为 `latex` 时三种来源均可 |
| 判分方式 | `choose`、`judge` 比对答案；`blank` 按文本归一化或数值容差；`short` 按采分点或专家评阅 |

## 附录 A　最小文件骨架

```yaml
version: 1
meta:
  title: "示例题库"
  subject: 物理
  grade: 高三
  authors: [示例作者]
  updated: "2026-11-21"
content:
  math: auto
  chemistry: none
  answer_input: text
questions:
  - number: 1
    points: 2.0
    type: judge
    stem: '滑动摩擦力的大小一定等于 $\mu mg$。'
    answer: false
```

题组写法：

```yaml
version: 1
questions:
  - number: 2
    points: 4.0
    material: '阅读材料。'
    members:
      - type: choose
        points: 2.0
        stem: '下列正确的是：'
        options: [甲, 乙, 丙, 丁]
        answer: 1
      - type: judge
        points: 2.0
        stem: '该判断为假。'
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
