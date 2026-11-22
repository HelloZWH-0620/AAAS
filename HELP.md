# AAAS 题库文件编写指南

**摘要** 本文规定 AAAS v1 题库交换文件的组成、字段取值与内容表达方式，并给出从空文件到可发布题目的编写流程。文件为 UTF-8 编码的 YAML，由版本号和题目列表构成；每道题自包含，可独立抽出并保留卷面题号。题目分有小题与无小题两种形态，前者细节写在每个 item，后者细节直接写在题目层。正文支持 Markdown、LaTeX、化学式、插图与表格，四种能力以插件按题声明。文中给出四题型的答案文法、字段表、机器校验项与最小文件骨架。

**关键词** 题库交换格式；YAML；JSON Schema；插件；评分方式

## 1 引言

AAAS（A Adaptive Assessment Schema）用于在不同出题、组卷、作答、评分软件之间交换单道题目。设计目标有三点：一道题脱离整卷后仍然完整；字段可被机器校验；评分所需信息随题目一起传递。

本文面向编写题库文件的人。读者应能据此独立写出合规 YAML。文件命名、编码与 Schema 声明见 [README.md](./README.md)。规范的章节组织与 Schema 编排参照 CSES [6]。

## 2 术语与记号

| 记号 | 含义 |
| --- | --- |
| 题目层 | `questions[]` 中的一项，承载科目、作者、时间、版本、分值、引导语、来源与插件声明 |
| 细节层 | 题型、知识点、分值、题目内容、答案、评分方式、正则给分点、标准答案、评分细则九个字段 |
| item | 小题，`items[]` 中的一项，承载一套细节层字段 |
| 插件 | math / chemistry / photo / excel 之一，按题声明，定义见同名 `*.schema.json` |
| `{{n}}` | 题目内容中的空位编号，n 从 1 起连续 |
| tenths | 修订版本的整数表示，10 表示 1.0，每次修订加 1 |

类型记号：`string` 文本，`number` 数值，`int` 整数，`bool` 布尔，`list` 列表，`map` 键值表，`enum` 封闭枚举。`null` 表示留空。

## 3 文件组成

顶层只有两个字段。

**表 1　顶层字段**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `version` | int | 是 | 固定为 1 |
| `questions` | list | 是 | 题目列表，至少一项，每项为一道题 |

## 4 字段定义

### 4.1 题目层

**表 2　题目层字段**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `subject` | string | 是 | 科目 |
| `authors` | string[] | 是 | 出题人，按署名顺序，至少一人 |
| `date` | string | 是 | 出题时间，格式 `yyyy.mm.dd`，须加引号 |
| `revision` | int | 是 | 修订版本 tenths，最小 10（即 1.0），每次修订加 1 |
| `points` | number | 是 | 分值，0.5 的整数倍；有小题时等于各 item 分值之和 |
| `instruction` | string | 是 | 引导语，如「阅读材料，完成下列问题」 |
| `source` | string | 是 | 来源，纯文本，如「金太阳联考」 |
| `plugins` | map[] 或 null | 否 | 启用的插件列表，每项见第 5 节；留空写 null |
| `number` | int | 否 | 自定义大题题号，整卷抽出时保留原号；省略则按出现顺序编号 |
| `items` | list | 否 | 小题列表；写出后题目层不得再写 `type` 与 `stem` |

### 4.2 细节层

**表 3　细节层字段**（有小题时写在每个 item，无小题时直接写在题目层）

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `type` | enum | 否 | `choose` / `blank` / `judge` / `short` |
| `knowledge` | string[] 或 null | 否 | 知识点列表；留空写 null |
| `points` | number | 否 | 分值，0.5 的整数倍 |
| `stem` | string | 否 | 题目内容，可内联插件，见第 5 节 |
| `answer` | 依题型 | 否 | 机器判分用的答案，见 6.1；留空写 null |
| `scoring` | enum | 否 | `auto` 自动 / `smart` 智能 / `manual` 人工 |
| `regex_points` | list 或 null | 否 | 正则匹配给分点，见 6.2；留空写 null |
| `standard_answer` | string 或 null | 否 | 标准答案文字，供智能与人工评分参考 |
| `rubric` | string 或 null | 否 | 评分细则，人工与智能评分按此逐点裁定 |

题目层必填的只有表 2 前七项。表 3 字段全部可选，允许留空：这就是「每题写全九个字段，部分字段为空」的写法——字段都写出来，不适用的填 null 或空串。

题目层一旦写出 `items`，就不得再写 `type` 与 `stem`，避免细节出现在两层造成归属不明。Schema 对这两项做了互斥校验。

### 4.3 正则给分点

**表 4　`regex_points[]` 元素**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `pattern` | string | 是 | 正则表达式，匹配考生作答文本 |
| `point` | string | 否 | 给分点名称，用于输出判定依据 |
| `score` | number | 否 | 该点分值，0.5 的整数倍 |

## 5 内容表达

### 5.1 内联插件语法

题目内容 `stem` 是字符串，普通文字按 Markdown 解析。需要特殊排版时用成对标记包裹内容，标记只在本题 `plugins` 已声明该插件时有效。

| 标记 | 插件 | 内容写法 |
| --- | --- | --- |
| `{{math}}…{{/math}}` | math | LaTeX 源码，行内 `$...$` 或块级 `$$...$$` |
| `{{chem}}…{{/chem}}` | chemistry | 化学方程式，如 `\ce{2SO2 + O2 <=> 2SO3}` |
| `{{photo}}…{{/photo}}` | photo | 图号或图片引用，如 `图1 受力分析图` |
| `{{excel}}…{{/excel}}` | excel | GFM 表格，单元格内可放 `{{n}}` 空位 |

空位 `{{n}}` 写在 `stem` 中，n 从 1 起连续，个数等于 `answer` 列表长度。带空位的表格必须写在 `stem`，不能写在材料里——材料会被多题共享，空位无法归属。

### 5.2 插件声明

`plugins` 每项含 `name` 与插件专属字段。`name` 为 `math` / `chemistry` / `photo` / `excel` 之一，专属字段见对应 Schema。未声明的插件不得在 `stem` 中使用。

**表 5　插件专属字段**

| 插件 | 字段 | 取值 | 默认 |
| --- | --- | --- | --- |
| math | `engine` | katex / mathjax | katex |
| math | `inline` / `block` | 定界符字符串 | `$` / `$$` |
| math | `packages` | 字符串列表 | 空 |
| chemistry | `engine` | mhchem / none | mhchem |
| chemistry | `structure` | smiles / svg / none | none |
| photo | `format` | svg / webp / png / jpeg | svg |
| photo | `min_dpi` | 整数 ≥ 150 | 300 |
| photo | `allow_upload` | bool | true |
| photo | `max_upload_mb` | 正数 | 10 |
| excel | `engine` | gfm / html | gfm |
| excel | `max_rows` / `max_cols` | 正整数 | 30 / 12 |
| excel | `allow_merge` | bool | false |

`photo.allow_upload` 控制考生能否上传 png 作答。数学公式以 LaTeX 源码存入字段，排版参数由渲染器在读取时确定；需要随字号、版心与主题自适应时，不要改用固定尺寸的图片。

### 5.3 Markdown 与 YAML 的交界

LaTeX 中的反斜杠在 YAML 双引号里会被当转义符，`\frac` 必须写成 `\\frac`，`\,` 更是直接非法。含 LaTeX 的字符串一律用单引号或 `|-` 块标量。以 `#` 开头的行是 YAML 注释，正文里的井号需放进引号。

## 6 题型与评分

### 6.1 答案文法

**表 6　四题型的答案形态**

| 题型 | `answer` 类型 | 文法 | 判分 |
| --- | --- | --- | --- |
| `choose` | int 或 int[] | 序号从 1 起；多选为严格升序、无重复的列表 | 序号比对 |
| `blank` | list | 每项为字符串或 `{value, tol, unit}`；个数等于空位数 | 文本归一化或数值容差 |
| `judge` | bool | `true` 或 `false`，不接受字符串 | 布尔比对 |
| `short` | string 或 null | 参考答案；留空写 null | 走 `regex_points` 与 `rubric` |

数值对象 `{value, tol, unit}`：`value` 为数值答案，`tol` 为相对容差（`0.02` 表示 ±2%），`unit` 为单位。Schema 按 `type` 把答案形态绑定，`judge` 写成 `"true"` 会被拒绝。

「判断并改正」不是判断题，改用 `short`，把改正要求写进 `stem`，采分点写进 `regex_points` 或 `rubric`。

### 6.2 评分方式

**表 7　`scoring` 取值**

| 取值 | 依据字段 | 处理 |
| --- | --- | --- |
| `auto` | `answer` + `regex_points` | 正则逐点匹配，命中给分；无正则时仅比对 answer |
| `smart` | `standard_answer` + `rubric` | 模型比对作答给出初判分与理由，低置信度转人工 |
| `manual` | `rubric` | 教师逐点裁定，`regex_points` 仅作参考 |

png 作答先经 OCR 转为文本再进入上述流程，原图保留供复核。自动判分无法裁定或字段留空时，转人工。各小题得分求和为本题得分，留空字段不参与给分。

### 6.3 编写约束

- 判断题的 `stem` 为可判真假的单句陈述，错误项错在明确知识点，不靠荒诞表述。
- 简答题的 `stem` 限定答题角度与范围；`rubric` 按点列出，不写「略」。
- 分值取 0.5 的整数倍；`regex_points[].score` 之和宜等于该题 `points`。
- 题号 `number` 只在需要保留原卷号时写，其余省略。

## 7 编写流程

1. 建立文件，首行写 `version: 1`。
2. 写 `questions:`，逐道添加题目。
3. 填题目层七项必填字段：`subject`、`authors`、`date`、`revision`、`points`、`instruction`、`source`。
4. 判断本题是否有小题。无小题，进入第 5 步；有小题，写 `items:`，逐项填写细节层，进入第 6 步。
5. 无小题：在题目层写细节层九字段，不写 `items`。
6. 有小题：在题目层只写共享信息，不写 `type` 与 `stem`；每个 item 写细节层。
7. 在 `stem` 中写正文，用 `{{n}}` 标空位，用成对标记内联插件。
8. 按第 5.2 节在 `plugins` 声明所用插件，`photo.allow_upload` 按需开启。
9. 按题型填 `answer`，填 `scoring`，填 `regex_points`、`standard_answer`、`rubric`。
10. 用第 8 节的方法校验，修正后交付。

## 8 校验

### 8.1 Schema 校验

编辑器在文件首行加入：

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/HelloZWH-0620/AAAS/main/aaas.schema.json
```

命令行：

```
check-jsonschema --schemafile aaas.schema.json example.yaml
```

主 Schema 通过 `$ref` 引用同目录四个插件 Schema。用 Python 校验时，需把五个 Schema 按各自 `$id` 注册到 resolver 的 store，代码见 [README.md](./README.md) 所链接的示例约定。

Schema 覆盖：版本号、日期格式、版本下界、分值粒度、题型枚举、题型与答案绑定、选项序号唯一、插件名与插件字段、未知字段拒绝、题组与单题互斥。

### 8.2 人工自检

JSON Schema 无法表达跨字段求和与编号连续，以下需自行检查或在软件中检查：

- C1 有小题时，各 `item.points` 之和等于题目层 `points`。
- C2 `regex_points[].score` 之和不超过该题得分。
- C3 `{{n}}` 从 1 起连续，个数等于 `answer` 列表长度。
- C4 `stem` 中用到的插件都已在 `plugins` 声明。
- C5 `author` 顺序与署名一致；`date` 为真实出题日期。
- C6 同层 `number` 不重复；省略时按顺序生成。

## 9 作答数据

作答与出题共用同一套内容来源：直接输入、`.md` 文件、`.tex` 文件、png 上传。作答只写入题库给定的槽位（题号、小题号、`{{n}}`），不替换题干。

| 约束 | 内容 |
| --- | --- |
| 槽位固定 | 按题号与空位编号定位，不改题干、不新增空位 |
| 格式限制 | `photo.allow_upload` 为 false 时拒收 png；作答格式须符合已声明插件 |
| 判分方式 | 依 `scoring` 走对应流程，字段留空的小题转人工 |

## 附录 A 最小文件骨架

```yaml
version: 1
questions:
  - subject: 数学
    authors: [张三]
    date: '2026.11.22'
    revision: 10
    points: 2.0
    instruction: '完成下列问题。'
    source: 金太阳联考
    plugins: null
    number: 1
    type: blank
    knowledge: [函数单调性]
    stem: '函数 $f(x)=x^2$ 在 $x=1$ 处的值为 {{1}}。'
    answer:
      - value: 1
    scoring: auto
    regex_points: null
    standard_answer: '1'
    rubric: null
```

## 致谢

感谢 SmartTeachCN 的 CSES 规范在章节组织、Schema 拆分与文档编排上提供的参照。

## 参考文献

[1] JSON Schema Draft 07. https://json-schema.org/specification-links.html#draft-7

[2] YAML 1.2 Specification. https://yaml.org/spec/1.2/spec.html

[3] CommonMark Spec. https://spec.commonmark.org/

[4] KaTeX Documentation. https://katex.org/docs/supported.html

[5] mhchem Documentation. https://mhchem.github.io/MathJax-mhchem/

[6] SmartTeachCN. CSES: The Course Schedule Exchange Schema. https://github.com/SmartTeachCN/CSES
