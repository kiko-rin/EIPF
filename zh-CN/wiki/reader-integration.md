# 阅读器集成指南

> 实现一个 EIPF 阅读器需覆盖以下能力。阅读器负责解包、层级解析、资源查找与渲染器装载。

## 1. 文件加载

1. 用 ZIP 库加载用户选择的文件。
2. 构建 `path -> 内容` 映射（跳过目录）。
3. 判定层级：`series.json` > `album.json` > `index.json`，否则报错。
4. 读取标题：`meta.title`（l10n 对象，优先当前语言），回退文件名。

## 2. 三层文件模型

阅读器维护三份文件映射：

| 字段 | 层 | 说明 |
|---|---|---|
| Series 文件 | Series | 外层 ZIP 全量文件 |
| Album 文件 | Album | 当前 Album 解压后的文件 |
| Entry 文件 | Entry | 当前 Entry 解压后的文件 |

### 嵌套 ZIP 解压

- 加载 Album：从 Series 层取 `album.path` 对应条目，解压。
- 加载 Entry：先查 Album 层，再查 Series 层的 Entry ZIP；解压后若所有文件共享同一公共前缀则自动剥离。

## 3. 三级资源查找

任何资源按 Entry → Album → Series 顺序查找，命中即返回：

```
当前 Entry ZIP → 当前 Album ZIP → Series ZIP → 未找到
```

命中后转为 Blob / Data URL 返回给渲染层。

## 4. 渲染器装载

1. 三级查找 `renderer/manifest.json`，要求 `type === "embedded-renderer"`。
2. 找到 → 进入内嵌渲染器模式（见 `renderer-protocol.md`）。
3. 未找到 → 降级渲染模式（加载 `shared/scenario.js` / `scenario.css`）。

## 5. 渲染章节

1. 解压当前 Album 与 Entry。
2. 查找 `body.xhtml`（任意以 `body.xhtml` 结尾的文件）。
3. 装载渲染器或降级渲染。
4. 内嵌模式：发送 `reader:open`（正文资源已替换为本地可访问形式）。
5. 降级模式：组装 `<style>scenario.css</style>` + `<body>body.xhtml</body>` + `<script>scenario.js</script>` 写入 WebView / iframe。

## 6. 章节列表与推进

- 列表：Series 层 `albums[]` 展开 Album，点击后解压 Album 读取 `album.json.entries[]`。
- 推进：`renderer:ended` 后进入同 Album 下一 Entry → 下一 Album → 结束。

## 7. 通信要点

- 所有消息含 `source` 字段，接收方必须校验。
- 资源请求-响应用 `id` 匹配。
- 渲染器不直接访问文件系统；资源经 `renderer:resource` 请求，由阅读器响应。
- 进度：`renderer:state` 的 `currentIndex` 保存精确到条目的进度。

## 8. 可选安全（2.0.1-beta1）

打开文件时，若根 JSON 含 `security` 属性（见 `eipf-spec.md` §5），按 `enforcement` 执行检查：

| enforcement | 阅读器动作 |
|---|---|
| `signature` | 内置公钥验签 `contentHash`；失败 → 拒绝打开 |
| `device` | 校验 `security/license.json`（验签 + 设备令牌一致 + 未过期） |
| `encryption` | 资源读取时先解密再交付渲染层 |

- 不支持 `enforcement` 时应提示"需要更高版本阅读器"，不得降级放行。
- 设备令牌应由安全硬件（如 Android Keystore 不可导出密钥）派生，并可在设置中展示供打包器绑定。

## 9. 关键常量

- 共享 CSS 路径：`shared/scenario.css`。
- 共享 JS 路径：`shared/scenario.js`。
- 正文文件：`resource/text/body.xhtml`（以 `body.xhtml` 结尾匹配）。
- 渲染器入口：`renderer/<manifest.entry>`。
