# aaas 题库规范

`example.yaml` 是带注释的完整示例，本文讲怎么填、填完长什么样。

## 1. 三十秒上手

```yaml
questions:
  - number: 1
    points: 2.0
    items:
      - number: 1
        category: judge
        points: 2.0
        stem: '滑动摩擦力的大小一定等于 $\mu mg$。'
        answer: false
```

渲染出来是：一道 2 分的判断题，题干里 $\mu mg$ 排成公式，学生点“对 / 错”，系统自动判分。

## 2. 内容从哪来：三个来源，可混搭

| 来源 | 写法 | 适合 |
| --- | --- | --- |
| 用户自输入 | 字段直接写 | 短题干、选项、答案、采分点 |
| 上传 `.md` | `stem_ref: "items/q1-1.md"` | 长材料、图文混排、表格、需要排版 |
| 上传 `.tex` | `stem_ref: "items/q1-1.tex"` | 公式主导的题干与推导 |

三条规则：

1. **每个槽位独立选择。** 同一道题里，材料走 `.md`、题干内联、答案走 `.tex`，全部允许，最后拼成一道完整题目。
2. **槽位与本体二选一。** `stem` 和 `stem_ref` 同时出现算非法，不做“取其一”。
3. **只有正文字段能外链。** 可外链的是 `ask`、`material`、`stem`、`answer`、`explanation`；`options`、`key_points` 是结构字段，只能内联。

`.md` 按 Markdown 解析，里面可以写 `$...$`；`.tex` 整段按 LaTeX 渲染、不再解析 Markdown，所以不适合放大段中文——中文正文用 `.md`，纯公式推导用 `.tex`。

**老师编辑时同样可混搭**，规则与新建完全一致。只有一处差异：已有题目的槽位若是外链，改的是那个文件，YAML 不动；要把外链改成内联，需要“内联化”——读入文件内容写进字段、同时删掉 `_ref`；反向叫“外链化”——内容落成文件、字段换成 `_ref`。混搭的代价是同一题可能同时开着 `.md` 双栏编辑器和 `.tex` 公式编辑器，保存时各写各的。

## 3. 输入 → 输出

| 你写的 | 渲染成 | 用在哪 |
| --- | --- | --- |
| `$f=\mu mg\cos\theta$` | 与正文同字号的一行公式，变量斜体、`\cos` 正体、上下标自动缩小 | 题干、选项、解析 |
| `$$\Delta x=v_0t+\tfrac12at^2$$` | 独占一段、居中，分数展开成全尺寸 | 材料、解析 |
| `\ce{2SO2(g) + O2(g) <=> 2SO3(g)}` | 下标自动降位、可逆箭头加长，`<=>` 渲染成 ⇌ | 化学题干 |
| `$A\_B\_$` | 斜体 A、下划线占位、斜体 B，生物基因型的标准写法 | 生物题干 |
| `**{{1}}**` | 加粗包住的可输入空位，宽度按答案字数预留 | 填空 |
| GFM 表格里写 `{{n}}` | 该单元格变成输入框，其余单元格正常显示 | 化学三段式、实验数据表 |
| `![](mech/q1-force.svg)` | 独立成段插图，超出版心等比缩放，打印不糊 | 材料、题干 |
| `answer` 写 `{value: 10, tol: 0.02, unit: N}` | 学生答 9.9 或 10.1 都算对，答 10 m/s 判单位错 | 填空（理科） |
| `stem_ref: "q1.tex"` | 读文件后整段按 LaTeX 渲染 | 五个正文字段 |

两个容易踩的点：

- **`$` 不是永远都是公式。** 语文题干里的 `$5` 是美元符号。`content.math` 分三档：`latex` 强制、`auto` 成对才解析（默认）、`none` 全按字面量。
- **YAML 双引号里的 `\,` 会报错。** 含 LaTeX 的字符串一律用单引号或 `|-` 块标量，`"$2.5\,\mathrm{m/s^2}$"` 解析直接失败。

## 4. 表格是内容，不是题型

表格直接写在 `material` 或 `stem` 里，用 GFM 语法，不需要专门的题型：

```yaml
stem: |-
  测得数据如下：

  | 浓度 / ($\mathrm{mol\cdot L^{-1}}$) | $\ce{SO2}$ | $\ce{O2}$ | $\ce{SO3}$ |
  | --- | --- | --- | --- |
  | 起始 | $1.0$ | $0.5$ | $0$ |
  | 转化 | $0.8$ | {{1}} | $0.8$ |
```

五条规矩：

1. 单元格里可以放 `{{n}}`，该格渲染为输入框——题型仍然是 `blank`，答案统一放在 `answer` 列表里。
2. 空位编号**先行后列、先左后右**，跨行时继续递增。
3. 每行列数必须相同，表头列数须与数据行一致。
4. 空位不得跨单元格，一格建议只放一个。
5. **`{{n}}` 只能出现在 `stem` 里。** `material` 被多道小题共用，写在里面无法归属到具体小题；带空的表格必须放在 `stem`。

合并单元格 GFM 不支持，需要时改用白名单 HTML（`allow_html: tables`）或直接转图片。

## 5. 字段速查

| 路径 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `version` | int | 是 | 固定 `1` |
| `meta.title` | string | 否 | 题库名称 |
| `meta.subject` | string | 否 | 学科名，与 cses 的 `subjects[].name` 一致 |
| `meta.grade` | string | 否 | 年级，写中文（高一 / 七年级 / 高三） |
| `meta.authors` | string[] | 否 | 作者列表，按署名顺序 |
| `meta.updated` | string | 否 | `YYYY-MM-DD` |
| `content.default_source` | enum | 否 | `inline` / `file`，只影响编辑器默认表单，不参与校验 |
| `content.math` | enum | 否 | `latex` / `auto` / `none`，默认 `auto` |
| `content.chemistry` | enum | 否 | `mhchem` / `none`，默认 `none` |
| `content.answer_input` | enum | 否 | `text` / `latex`，默认 `text` |
| `content.allow_html` | enum | 否 | `false` / `tables` / `all`，默认 `tables` |
| `content.asset_dir` | string | 否 | 图片与 `_ref` 的基准目录，默认 `assets` |
| `category[].key` | string | 是 | 题型标识，全库唯一 |
| `category[].name` | string | 是 | 显示名 |
| `category[].auto_grade` | bool | 是 | 是否可自动判分 |
| `category[].fields` | string[] | 是 | 该题型专属必填字段 |
| `category[].answer_type` | string | 是 | `answer` 的取值形态，供校验 |
| `questions[].number` | int | 是 | 大题题号，≥1，本层唯一 |
| `questions[].points` | float | 是 | 大题分值，须等于各小题之和 |
| `questions[].ask` | string | 否 | 引导语；可改用 `ask_ref` 外链 |
| `questions[].material` | string | 否 | 材料；可改用 `material_ref` 外链 |
| `questions[].tags` | string[] | 否 | 知识点标签，与小题取并集 |
| `questions[].origin` | map | 否 | 来源，见下方 origin 段 |
| `questions[].items` | list | 是 | 小题列表 |
| `items[].number` | int | 是 | 小题题号，≥1，本大题内唯一 |
| `items[].category` | enum | 是 | 命中 `category[].key` |
| `items[].points` | float | 是 | 小题分值，0.5 的整数倍 |
| `items[].stem` | string | 是 | 题干；可改用 `stem_ref` 外链，二者二选一 |
| `items[].answer` | 依题型 | 是 | 见第 6 节；可改用 `answer_ref` 外链 |
| `items[].options` | string[] | 选择题必填 | 2~8 项，只允许行内 Markdown |
| `items[].key_points` | list | 否 | 简答采分点，`{point, score}` |
| `items[].max_words` | int | 否 | 简答作答字数上限 |
| `items[].explanation` | string | 否 | 解析；可改用 `explanation_ref` 外链 |
| `items[].tags` | string[] | 否 | 知识点标签，省略则继承大题 |
| `items[].origin` | map | 否 | 来源，省略则继承大题 |
| `origin.type` | enum | 块内必填 | `human` / `ai` / `adapted` / `excerpt` |
| `origin.source` | string | 否 | 出处自由文本 |
| `origin.ref` | map | 否 | `{number, url}`，结构化定位原题 |
| `origin.model` | string | `ai` 必填 | 生成模型标识 |
| `origin.generated_at` | string | `ai` 必填 | `YYYY-MM-DD` |
| `origin.reviewed` | bool | `ai` 必填 | 是否已人工审校，`false` 不进正式练习 |
| `origin.reviewed_by` | string | 否 | 审校人 |

## 6. 四种题型

| key | 名称 | 专属必填 | 判分 | 关键约束 |
| --- | --- | --- | --- | --- |
| `choose` | 选择题 | `options` `answer` | 自动 | 2~8 项；`answer` 序号从 1 起；多选写 `[1,3]` |
| `blank` | 填空题 | `answer` | 自动 | `{{n}}` 从 1 连续；答案可写文本或 `{value,tol,unit}` |
| `short` | 简答题 | `answer` | 采分点 / 专家 | `key_points` 各项之和必须等于本题 `points` |
| `judge` | 判断题 | `answer` | 自动 | 单句陈述；只能 `true` / `false` |

## 7. 学科适配

| 学科 | 建议配置 | 要点 |
| --- | --- | --- |
| 数学 | `math: latex` | 多行用 `\begin{aligned}`；公式编号写正文；几何图用 SVG |
| 物理 | `math: auto` | 单位正体并用 `\,` 分隔；矢量 `\vec{F}`；受力图 SVG |
| 化学 | `math: auto`、`chemistry: mhchem` | 方程式一律 `\ce{}`；结构式、装置图用 SVG（mhchem 只支持线性式）；有效数字要卡时答案用字符串而非数值对象 |
| 生物 | `math: auto` | 基因型写 `$AaBb$`（斜体）；杂交 `\times`；系谱图、遗传图解用 SVG |
| 语文 / 英语 / 政史地 | `math: none` | `$` 按字面量，避免 `$5` 被误判为公式开头 |

理综、文综这类合卷用 `math: auto` + `chemistry: mhchem`（仅当含化学题），不要用 `latex` 强制——生物和物理混排时落单的 `$` 会直接报错。

## 8. 校验清单

发布前按顺序跑，前一条不通过就阻断：

1. `version` 等于 `1`；`category`、`questions` 存在。
2. 小题的 `category` 命中注册表，`fields` 列出的字段齐全。
3. 大题 `points` = 各小题 `points` 之和。
4. `key_points` 的 `score` 之和 = 该小题 `points`。
5. `{{n}}` 从 1 连续、无重复，个数 = `answer` 长度。
6. `{{n}}` 只出现在 `stem` / `stem_ref` 里，不在 `material`、`ask`、`explanation`、`answer` 中。
7. `choose` 的 `answer` 在 `1..len(options)` 内且不重复；`judge` 的 `answer` 是布尔。
8. 每个槽位的本体与 `_ref` 不同时出现；`options` / `key_points` 没有 `_ref`。
9. GFM 表格各行列数一致；含 `{{n}}` 的表格位于 `stem`。
10. `_ref` 与图片路径存在、不越出 `asset_dir`、UTF-8 编码。

第 10 条在草稿态降为警告，发布态为错误。

## 9. 作答侧：同一套逻辑

学生提交解答走一样的三个来源：**直接输入 / 上传 `.md` / 上传 `.tex`**。约束比出题侧紧三条：

1. **槽位不能改。** 学生只能往题库给定的位置上填：`questions[].number`、`items[].number`、单元格里的 `{{n}}`。不能整段上传文件替换题干。
2. **由题库决定能否用 LaTeX。** `content.answer_input` 为 `text` 时，作答侧拒收 `.tex`，只收纯文本；为 `latex` 时三种来源都收。
3. **判分方式跟着题型走。** `blank` 按文本归一化或数值容差比对；`short` 按采分点或交专家；`judge` / `choose` 直接对答案。

所以“学生上传解答和出题一个逻辑”成立，但**自由度更低**：出题侧可以自由选择每个槽位的来源，作答侧只能在既定槽位里选来源。

## 10. 为什么 LaTeX 能自动适配，图片不能

**LaTeX 是描述结构的源码，排版发生在渲染时；图片是已经定型的像素。**

- AI 写 `\frac{a}{b}` 只声明“a 是分子”，字号、线宽、间距由 KaTeX 当场算，所以父容器字号、版心宽度、缩放比、打印 DPI 都能作为输入。
- 行内 `$...$` 遇到分数自动压成小号，块级 `$$...$$` 才展开全尺寸；图片没有这层语义，只能等比缩放，放大就模糊。
- 字色继承父级 `color`，深色模式自动反白；字号用 `em`，正文 12pt 公式就是 12pt。
- 附带：可搜索、可复制成源码、可 diff、能输出 MathML 给屏幕阅读器，同一份内容能喂网页、小程序、PDF。

代价是依赖引擎和字体包，且 KaTeX 不支持自定义宏与 `\label` / `\ref`，所以公式编号写在正文里。
