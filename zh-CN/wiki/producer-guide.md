# 制作指南：生成 EIPF 文件

> 本指南描述如何把内容源制作成 EIPF 文件。内容源的具体格式由制作者自行决定，本指南只规定 EIPF 侧的结构约束。

## 1. 前提

- 具备内容源：各作品的脚本 / 素材（文本、图片、音频、视频）。
- 一个可用的打包工具或脚本：解析内容源 → 生成 `body.xhtml` → 组装三层 ZIP。
- 资源映射：把内容源中的资源标识解析为可下载的 URL。

## 2. 打包流程（建议 5 个阶段）

| 阶段 | 说明 |
|---|---|
| Phase 1 | 解析每个场景 → 生成 body.xhtml → 抽取资源列表 → 汇总资源路径映射 |
| Phase 2 | 下载全部资源（失败重试 ≤3，支持 URL 大小写回退） |
| Phase 3 | 构建每个 Entry ZIP → 组装 Album ZIP → 收集 Series album 元数据 |
| Phase 4 | 复制 `renderer/` 与 `shared/`（scenario.css / scenario.js） |
| Phase 5 | 写入 `series.json`、渲染清单，最终打包 `.eipfs` |

## 3. 生成规则速查

### Entry ZIP 内容（建议 `ZIP_STORED`）

- `index.json`
- `main.xml`
- `META-INF/manifest.xml`
- `resource/text/body.xhtml`
- `resource/{rtype}/{filename}`（引用到的资源）

### Album ZIP 内容（`ZIP_STORED`）

- `album.json`
- `entries/<id>.eipf`（每个 Entry 一个嵌套 ZIP）

### Series ZIP（外层，文本 Deflate / 二进制 Store）

- `series.json`
- `renderer/`
- `shared/scenario.css`、`shared/scenario.js`
- `resource/render/manifest.json`（可选）
- `albums/<id>.eipfa`

## 4. ID 生成

- Entry / Album ID：对源路径或标题做清洗（去除 `<>:"/\|?*` 与空白，替换为 `_`），保证唯一且为合法文件名。
- 专辑标题：由制作者按内容分类生成。

## 5. 资源分类与下载

分类：`audio` / `backgrounds` / `characters` / `illustrations` / `video`。

下载策略：

- 目标路径 `resource/{rtype}/{filename}`。
- 文件已存在则跳过。
- 失败重试：仅 5xx、超时、连接错误可重试，最多 3 次。
- 大小写回退：原链接失败后用 URL 小写化重试。

## 6. 校验

每个文件计算 `sha256:` 前缀校验和，写入 `META-INF/manifest.xml`，可据此校验完整性。

## 7. 可选安全保护（2.0.1-beta1）

如需版权保护，可在打包时启用 `security` 属性（见 `eipf-spec.md` §5）：

- **signature**：制作者私钥对内容哈希签名，防篡改 / 防伪。
- **device**：注入 `security/license.json` 绑定目标设备令牌，防复制分享。
- **encryption**：用 AES-256-GCM 加密 `resource/` 与 `renderer/`，防解包看内容。

实现建议：

- 密钥对（RSA-2048）：制作者生成私钥自留，公钥内嵌进阅读器。
- 设备令牌：由阅读器安全硬件派生，用户在"设置"中复制后填入打包器。
- 注意：客户端侧校验属"提高成本"层面，无法绝对阻止专业逆向。
