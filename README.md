<div align="center">

# AAAS 通用题库格式

在不同出题、组卷、作答软件之间交换整份题库

</div>

> [!NOTE]
> AAAS v1 为现行版本。后续扩展以新增可选字段为主，已写好的 v1 文件不需要改写。

## 简介

AAAS 是一种通用的题库交换格式，用于在不同软件之间交换整份试卷——题型注册表、材料、题干、选项、答案、采分点、来源标注，全部写在一个 YAML 文件里。

设计目标是简单、易读、可机器校验：文件本身只存数据，渲染、判分、组卷都交给读它的软件；`meta.subject` 与 [CSES](https://github.com/SmartTeachCN/CSES) 的 `subjects[].name` 对齐，题库与课程表能挂在同一套学科名下。

## 特点

- **简单**：一份 YAML 写完整卷。子题数、选项数、空位数全部由内容长度推导，没有需要手工同步的计数字段。
- **通用**：正文用 Markdown，公式用 LaTeX，化学式用 mhchem，插图用 SVG / PNG；数学、理综、文科卷共用同一套结构。
- **可混搭**：每个正文字段既能内联，也能外链 `.md` / `.tex`。同一道题里材料外链、题干内联、答案外链同时存在也没问题。
- **可校验**：提供 [JSON Schema](./aaas.schema.json)，编辑器里边写边补全，提交前能自动查错。

## 代码补全

引入 [Schema 文件](https://raw.githubusercontent.com/HelloZWH-0620/AAAS/main/aaas.schema.json)，
并进行以下配置来启用补全与校验。

在 YAML 首行加一行，多数编辑器会自动识别：

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/HelloZWH-0620/AAAS/main/aaas.schema.json
```

- [VS Code](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml)
- [JetBrains IDE](https://www.jetbrains.com/help/idea/yaml.html#use-schema-keyword)
- [Zed Editor](https://zed.dev/docs/languages/yaml#schemas)

## 要求

- 文件名: `xxx.yml` / `xxx.yaml`
- 编码: UTF-8
- 版本: `version: 1`

## 示例

见 [example.yaml](./example.yaml)。

字段逐个怎么填、渲染成什么样、发布前查什么，见 [HELP.md](./HELP.md)。

## 协议

[Apache-2.0](./LICENSE)
