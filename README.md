<div align="center">

# AAAS 通用题库格式

在不同出题、组卷、作答软件之间交换单道题目与题库

</div>

> [!NOTE]
> AAAS v1 为现行版本。后续扩展以新增可选字段为主，已写好的 v1 文件不需要改写。

## 简介

AAAS 是一种通用的题库交换格式，用于在不同软件之间交换题目。每道题自包含，可独立抽出并保留卷面题号；也可选用题组形态共享一段材料。文件包含题型、题干、选项、答案、采分点与来源标注。

设计目标是简单、易读、可校验。

## 特点

- **以题为单位**：一道题自包含，可单独抽出并保留题号；题组作为可选形态。
- **简单**：选项数、空位数由内容长度推导，没有需要手工同步的计数字段。
- **通用**：正文 Markdown，公式 LaTeX，化学式 mhchem，插图 SVG / PNG；数学、理综、文科卷共用同一套结构。
- **可混搭**：每个正文字段既能内联，也能外链 `.md` / `.tex`。
- **可校验**：提供 [JSON Schema](./aaas.schema.json)，题型与答案形态的匹配在编辑器中即可检查。

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

详细说明，请见 [HELP.md](./HELP.md)。

## 协议

[Apache-2.0](./LICENSE)
