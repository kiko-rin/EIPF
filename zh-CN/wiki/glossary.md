# 术语表 / Glossary / 用語集

| 术语 | 说明 |
|---|---|
| **EIPF** | Electric Interactive Publications Format，基于 ZIP 的交互式数字出版物格式 |
| **Series（.eipfs）** | 最外层容器，含 `series.json`、渲染器、共享引擎与多个 Album |
| **Album（.eipfa）** | 中层容器，含 `album.json` 与多个 Entry |
| **Entry（.eipf）** | 最小内容单元，含 `index.json`、`main.xml`、清单与 `body.xhtml` |
| **body.xhtml** | Entry 的内容正文，以 `scene-entry` 线性条目承载内容 |
| **scene-entry** | 内容条目类型：dialog / Image / music / char / decision / sticker / controller 等 |
| **char-slot** | `char` 条目内部的角色槽位，含角色 ID、立绘路径与画布位置比例 |
| **bgm-change** | 独立于条目的音乐变化记录 |
| **manifest.xml** | Entry 的 `META-INF/manifest.xml` 文件清单，含 sha256 校验 |
| **renderer/manifest.json** | 内嵌渲染器清单，`type` 固定为 `embedded-renderer` |
| **内嵌渲染器** | EIPF 自带的 Web 应用，负责将 body.xhtml 渲染为交互页面 |
| **三级查找** | 资源按 Entry → Album → Series 顺序查找的机制 |
| **IPC / postMessage 协议** | 阅读器与渲染器之间的通信协议（见 `renderer-protocol.md`） |
| **桥接层** | 阅读器注入的脚本，把渲染器的 `postMessage` 转发给宿主，并暴露 `__eipfInit/Open/Resource/Next/Prev/Config/State/Destroy` |
| **startIndex** | `reader:open` 的续读目标条目索引（渲染器 `currentIndex`） |
| **低刷新优化（eink）** | 通过 `reader:init/config` 关闭渲染器打字机与动画，减少屏幕刷新 |
| **降级渲染模式** | 无内嵌渲染器时，用 `shared/scenario.js` / `.css` 直接渲染 |
| **sharedResources** | series.json / album.json 中指向共享引擎与渲染器的路径配置 |
| **l10n 对象** | 多语言字段，形如 `{ "zh-CN": "..." }` |
| **formatVersion** | 规范版本标识，当前 `"2.0.1-beta1"`（兼容 `"2.0.0"`） |
| **security** | 可选安全属性（签名 / 设备绑定 / 加密），见 `eipf-spec.md` §5 |
