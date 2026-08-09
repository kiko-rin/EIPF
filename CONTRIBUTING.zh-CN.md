# 为 EIPF 贡献

感谢您对 **EIPF（Electric Interactive Publications Format）** 的关注与贡献。

本仓库是一个**文档与规范项目**——包含规范性 Markdown 文档与 JSON Schema。请在发起 issue 或 pull request 前阅读本指南。

> English: [CONTRIBUTING.md](CONTRIBUTING.md) ／ 日本語: [CONTRIBUTING.ja.md](CONTRIBUTING.ja.md)

## 贡献方式

- **报告问题**：文档或 Schema 中的错误（示例错误、失效链接、表述歧义）。
- **提议规范变更**：新字段、新 `scene-entry` 类型、协议变更。
- **改进翻译**：中文 / English / 日本語 文档。
- **增加示例或工具**：校验器、生成器、阅读器。

## 开始

1. Fork 本仓库并克隆。
2. 创建分支：`git checkout -b fix/描述性名称`。
3. 进行修改。
4. 运行本地检查（见[检查](#检查)）。
5. 以清晰的提交信息提交并创建 pull request。

## 规范变更流程

本规范是**规范性**的——变更会影响制作者与阅读器。请遵循以下流程：

1. 先开一个 **issue**，描述问题与提议的变更。
2. 讨论影响：是否破坏现有包？是否向后兼容？
3. 同步更新受影响文档的**全部语言版本**（中文 / English / 日本語）。
4. 若涉及元数据，同步更新 `schema/` 下的对应 **JSON Schema**。
5. 在 **CHANGELOG** 中登记，并按语义化版本升级：
   - 修订（`2.0.1`）：澄清、修复、非破坏性新增。
   - 次版本（`2.1.0`）：向后兼容的功能新增。
   - 主版本（`3.0.0`）：破坏性变更。

## 翻译规则

各语言目录（`zh-CN/`、`en/`、`ja/`）保持相同结构：

```
<lang>/
├── README.md
├── spec/       # 规范文档
└── wiki/       # 指南与术语表
```

- 各语言保持**相同结构与链接**。
- **代码块、JSON 示例、文件路径、消息名**为语言无关内容，**禁止翻译**。
- 规范文档变更时，保持各语言目录同步。

## 检查

提交前请本地运行：

```bash
# 校验所有 JSON Schema 可解析
python -c "
import json, glob
for f in glob.glob('schema/*.json'):
    json.load(open(f, encoding='utf-8'))
print('all schemas valid')
"
```

## 提交信息风格

- 使用祈使句：`Add data-w to char-slot spec` / `Fix broken link in EN spec`。
- 作用域前缀：`spec:`、`schema:`、`docs:`、`i18n:`。
- 示例：`i18n: translate body-xhtml.md to ja`

## 行为准则

所有参与者须遵守[行为准则](CODE_OF_CONDUCT.md)。

## 许可

提交即表示您同意您的贡献以 [CC0 1.0 公有领域贡献（Public Domain Dedication）](LICENSE) 发布。
