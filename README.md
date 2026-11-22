<div align="center">

# AAAS 通用题库格式

在不同出题、组卷、作答、评分软件之间交换单道题目

[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](./LICENSE)

---

**注意** 本文为规范入口。字段怎么填见 [HELP.md](./HELP.md)，可运行示例见 [example.yaml](./example.yaml)，机器校验见 [aaas.schema.json](./aaas.schema.json)。

</div>

## 简介

AAAS 是一种题库交换格式，以**单道题目**为基本单元。科目、出题人、出题时间、修订版本、分值、引导语、来源写在题目层；题型、知识点、题干、答案、评分方式、正则给分点、标准答案、评分细则写在题目层或小题层。一道题可从整卷中抽出，并保留原卷面题号。

正文使用 Markdown，公式使用 LaTeX，化学式使用 mhchem，插图使用 SVG/PNG，表格使用 GFM。math、chemistry、photo、excel 四种能力以插件形式按题声明，各自有独立 Schema，由主 Schema 通过 `$ref` 引用。

## 特点

- **以题为单位**：每道题自包含，抽出后仍是可用的完整题目，不依赖整卷上下文。
- **小题可选**：无小题时细节写在题目层；有小题时写在每个 item，题目层只放共享信息。
- **评分可落地**：每题声明 auto / smart / manual 之一，配套正则给分点、标准答案、评分细则，直接对接阅卷流程。
- **插件按需加载**：四种插件各自独立，题目只声明用到的部分；`$ref` 让各插件可独立演进。
- **可校验**：提供 draft-07 Schema，题型与答案形态绑定，分值、日期、版本、插件字段均有约束。

## 文件组成

| 文件 | 用途 |
| --- | --- |
| [aaas.schema.json](./aaas.schema.json) | 主 Schema，draft-07 |
| [math.schema.json](./math.schema.json) | 数学公式插件 |
| [chemistry.schema.json](./chemistry.schema.json) | 化学方程式插件 |
| [photo.schema.json](./photo.schema.json) | 插图与 png 上传插件 |
| [excel.schema.json](./excel.schema.json) | 表格插件 |
| [example.yaml](./example.yaml) | 带注释的完整示例 |
| [HELP.md](./HELP.md) | 字段说明与编写流程 |

## 要求

文件扩展名 `.yaml` 或 `.yml`，UTF-8 编码，首行为 `version: 1`。

## 示例

```yaml
version: 1
questions:
  - subject: 物理
    authors: [张三]
    date: '2026.11.22'
    revision: 10
    points: 4.0
    instruction: '完成下列问题。'
    source: 金太阳联考
    plugins:
      - name: math
    type: judge
    knowledge: [摩擦力方向]
    stem: '滑动摩擦力的方向总是与物体的运动方向相反。'
    answer: false
    scoring: auto
```

## 校对与校验

用支持 JSON Schema 的编辑器打开 YAML 文件时，在文件头加入：

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/HelloZWH-0620/AAAS/main/aaas.schema.json
```

命令行校验与字段说明见 [HELP.md](./HELP.md)。

## 协议

Apache License 2.0，见 [LICENSE](./LICENSE)。
