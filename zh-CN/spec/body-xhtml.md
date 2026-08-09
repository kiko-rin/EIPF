# body.xhtml 内容格式 v2.0.1-beta1

> body.xhtml 是 Entry 的内容数据层，以**线性的 `scene-entry` 条目序列**承载内容。

## 1. 文档骨架

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="zh-CN">
<head>
  <meta charset="UTF-8"/>
  <title>第一章 开场</title>
  <link rel="stylesheet" href="shared/scenario.css"/>
</head>
<body class="scenario-body">
  <!-- page: entry_001 -->
  <!-- title: 第一章 开场 -->

  <!-- ═══ entries (index order) ═══ -->
  ...scene-entry 条目...

  <!-- ═══ bgm changes ═══ -->
  ...bgm-change 条目...
</body>
</html>
```

- 合法 XHTML 5，UTF-8，禁止 BOM。
- 每个条目一个 `<div class="scene-entry ...">`。
- 资源路径相对于 EIPF 根目录。
- 可选注入 `<script src="shared/scenario.js">`。

## 2. 条目类型（`data-type`）

本规范为**通用内容格式**：下表「来源命令」仅示意语义；不同作品的具体命令名由制作者自行映射到这些 `data-type`。

| `data-type` | 来源命令（示意） | 说明 |
|---|---|---|
| `dialog` | 说话者文本 / 纯文本 | 对话，含说话者与文本 |
| `Image` | 背景/图片命令 | 背景 / 图片 |
| `music` | 音乐命令 | 音乐（含引入与循环） |
| `char` | 角色命令 | 角色，含多个槽位 |
| `charaction` | 角色动作命令 | 角色动作 / 表情切换 |
| `theater` | theater | 影院 / 宽屏字幕模式 |
| `decision` | 选择命令 | 选择分支 |
| `sticker` | 字幕命令 | 居中文字 / 字幕 |
| `delay` | 延时命令 | 停顿 / 延时 |
| `blocker` | 色罩命令 | 屏幕闪白 / 闪黑 / 色罩 |
| `effect` | 滤镜命令 | 相机滤镜 |
| `shake` | 震动命令 | 镜头震动 |
| `predicate` | 分支谓词命令 | 决策分支谓词 |
| `navigate` | 跳转命令 | 剧情跳转 |
| `battle` | 战斗命令 | 进入战斗 |
| `tutorial` | 教程命令 | 教程触发 |
| `video` | 视频命令 | CG 视频 |
| `curtain` | 转场命令 | 黑幕转场 |
| `controller` | 其余控制命令 | 分页 / 元信息等 |

> 每个条目均含 `data-cmd`（命令名，如 `Blocker`）便于渲染器区分来源命令；新实现类型的参数以 `data-params`（JSON）完整保留。

### 2.1 dialog

```html
<div class="scene-entry scene-dialogue" data-type="dialog" data-index="0"
     data-speaker="角色甲" data-thought="true">
  <span class="speaker">角色甲</span>
  <span class="text">（内心独白）</span>
</div>
```

| 属性 | 说明 |
|---|---|
| `data-type` | `"dialog"` |
| `data-index` | 序号（从 0 递增） |
| `data-speaker` | 说话者（空 = 旁白） |
| `data-thought` | 内心独白标记 |

### 2.2 Image

```html
<div class="scene-entry scene-image" data-type="Image" data-index="1"
     data-ctrlcmd="[Image(image=bg_01)]" data-url="resource/backgrounds/bg_01.png">
</div>
```

| 属性 | 说明 |
|---|---|
| `data-type` | `"Image"`（首字母大写） |
| `data-ctrlcmd` | 原始命令文本 |
| `data-url` | 背景 / 图片路径 |

### 2.3 music

```html
<div class="scene-entry scene-music" data-type="music" data-index="2"
     data-ctrlcmd="[PlayMusic(key=..., intro=...)]"
     data-url="resource/audio/intro.mp3" data-loop="resource/audio/loop.mp3">
</div>
```

| 属性 | 说明 |
|---|---|
| `data-url` | 引入（intro）音频路径 |
| `data-loop` | 循环（key）音频路径 |

### 2.4 char

```html
<div class="scene-entry scene-char" data-type="char" data-index="3"
     data-ctrlcmd="[Character(...)]" data-url="resource/characters/char_a_1.png">
  <div class="char-slot" data-name="char_a_1#4"
       data-image-id="char_a_1" data-suffix="#4"
       data-url="resource/characters/char_a_1.png"></div>
  <div class="char-slot" data-name="char_b_1#0"
       data-image-id="char_b_1" data-suffix="#0"
       data-url="resource/characters/char_b_1.png"></div>
</div>
```

- `data-url`（char 条目自身）为当前聚焦角色的立绘路径。
- 内部一个或多个 `.char-slot` 描述具体角色：

| 属性 | 说明 |
|---|---|
| `data-name` | 完整角色 ID（`image_id + suffix`） |
| `data-image-id` | 图片 ID（`#suffix` 之前部分） |
| `data-suffix` | 后缀（如 `#4`，无则为空） |
| `data-url` | 立绘路径 |
| `data-x` | 角色中心 X（**画布宽度比例 0~1**） |
| `data-h` | 角色高度（**画布高度比例 0~1**） |
| `data-w` | 角色宽度（**画布宽度比例 0~1**，可选） |

> **画布约定**：制作者以固定画布（如 1280×720 px）推算角色位置/大小，本规范统一换算为**画布比例**。阅读器按自身视口等比缩放即可在任何分辨率下正确显示。
> 槽位 → `data-x` 的取值与语义由制作者定义（如按槽位序号映射）；默认单人居中 `0.50`，`data-h` 默认 `0.90`。

### 2.5 decision

```html
<div class="scene-entry scene-decision" data-type="decision" data-index="4"
     data-ctrlcmd="[Decision(options=...;..., values=...;...)]">
  <button class="choice-btn" data-choice-index="0">选项 A</button>
  <button class="choice-btn" data-choice-index="1">选项 B</button>
</div>
```

| 属性 | 说明 |
|---|---|
| `data-choice-index` | 选项序号 |

注：实际产出中按钮**不写** `data-target`，分支跳转由渲染器自定义实现（配合 `predicate`）。

### 2.6 sticker

```html
<div class="scene-entry scene-sticker" data-type="sticker" data-index="5"
     data-ctrlcmd="[Sticker(text=...)]" data-text="居中文字">
</div>
```

| 属性 | 说明 |
|---|---|
| `data-text` | 显示的文本 |

### 2.7 delay / blocker / effect / shake / predicate / navigate / battle / tutorial / interlude / cgitem / focusout / soundvolume / curtain 等

新实现类型统一以 `data-params`（JSON）保留全部参数，并附常用属性：

```html
<div class="scene-entry scene-delay" data-type="delay" data-index="10"
     data-ctrlcmd="[Delay(time=1.3)]" data-cmd="Delay" data-time="1300"
     data-params="{&quot;time&quot;: 1.3}"></div>

<div class="scene-entry scene-blocker" data-type="blocker" data-index="22"
     data-ctrlcmd="[Blocker(a=1, r=0, g=0, b=0, fadetime=2, block=true)]"
     data-cmd="Blocker"
     data-params="{&quot;a&quot;: 1, &quot;r&quot;: 0, &quot;g&quot;: 0, &quot;b&quot;: 0, &quot;fadetime&quot;: 2, &quot;block&quot;: true}"></div>

<div class="scene-entry scene-effect" data-type="effect" data-index="29"
     data-ctrlcmd="[CameraEffect(effect=&quot;Grayscale&quot;, ...)]"
     data-cmd="CameraEffect" data-effect="Grayscale"
     data-params="{&quot;effect&quot;: &quot;Grayscale&quot;, ...}"></div>

<div class="scene-entry scene-shake" data-type="shake" data-index="121"
     data-ctrlcmd="[CameraShake(duration=0.5, ...)]" data-cmd="CameraShake"
     data-params="{&quot;duration&quot;: 0.5, ...}"></div>

<div class="scene-entry scene-predicate" data-type="predicate" data-index="146"
     data-ctrlcmd="[Predicate(references=&quot;1&quot;)]" data-cmd="Predicate"
     data-references="1" data-params="{&quot;references&quot;: 1}"></div>

<div class="scene-entry scene-navigate" data-type="navigate" data-index="330"
     data-ctrlcmd="[SkipToThis]" data-cmd="SkipToThis"></div>

<div class="scene-entry scene-battle" data-type="battle" data-index="331"
     data-ctrlcmd="[StartBattle(stageId=&quot;stage_01&quot;)]" data-cmd="StartBattle"
     data-stage="stage_01" data-params="{&quot;stageId&quot;: &quot;stage_01&quot;}"></div>

<div class="scene-entry scene-tutorial" data-type="tutorial" data-index="332"
     data-ctrlcmd="[Tutorial(waitForSignal=&quot;guide_start&quot;)]" data-cmd="Tutorial"
     data-signal="guide_start" data-params="{&quot;waitForSignal&quot;: &quot;guide_start&quot;}"></div>

<div class="scene-entry scene-interlude" data-type="interlude" data-index="500"
     data-ctrlcmd="[interlude(maskid=&quot;m1&quot;,switch=true,channel=3)]" data-cmd="interlude"
     data-params="{&quot;maskid&quot;: &quot;m1&quot;, &quot;switch&quot;: true, &quot;channel&quot;: 3}"></div>

<div class="scene-entry scene-cgitem" data-type="cgitem" data-index="501"
     data-ctrlcmd="[cgitem(image=&quot;cg_01&quot;,layer=1,afrom=0,ato=1,aduration=1)]"
     data-cmd="cgitem" data-url="resource/illustrations/cg_01.png"
     data-params="{&quot;image&quot;: &quot;cg_01&quot;, &quot;layer&quot;: 1, &quot;afrom&quot;: 0, &quot;ato&quot;: 1, &quot;aduration&quot;: 1}"></div>

<div class="scene-entry scene-focusout" data-type="focusout" data-index="502"
     data-ctrlcmd="[focusout(duration=1,type=&quot;bg&quot;,from=0,to=0)]" data-cmd="focusout"
     data-params="{&quot;duration&quot;: 1, &quot;type&quot;: &quot;bg&quot;, &quot;from&quot;: 0, &quot;to&quot;: 0}"></div>

<div class="scene-entry scene-soundvolume" data-type="soundvolume" data-index="503"
     data-ctrlcmd="[SoundVolume(volume=1,channel=&quot;m&quot;,fadetime=1)]" data-cmd="SoundVolume"
     data-params="{&quot;volume&quot;: 1, &quot;channel&quot;: &quot;m&quot;, &quot;fadetime&quot;: 1}"></div>

<div class="scene-entry scene-curtain" data-type="curtain" data-index="504"
     data-ctrlcmd="[curtain(direction=0,fillfrom=0.1,fillto=0.25,fadetime=0.1)]" data-cmd="curtain"
     data-params="{&quot;direction&quot;: 0, &quot;fillfrom&quot;: 0.1, &quot;fillto&quot;: 0.25, &quot;fadetime&quot;: 0.1}"></div>
```

| 类型 | 关键属性 |
|---|---|
| `delay` | `data-time`（毫秒） |
| `blocker` | `data-params`（`a/r/g/b`、`fadetime`、`block`） |
| `effect` | `data-effect`（如 `Grayscale`） |
| `shake` | `data-params`（`duration/xstrength/ystrength/vibrato/randomness/fadeout`） |
| `predicate` | `data-references`（决策分支引用） |
| `navigate` | `data-cmd`（`SkipToThis` / `GotoPage`） |
| `battle` | `data-stage` |
| `tutorial` | `data-signal` |
| `interlude` | `data-params`（`channel/switch/maskid/size/offset` 等） |
| `cgitem` / `hidecgitem` | `data-url` + `data-params`（`layer/sfrom/sto/sduration/afrom/ato/aduration/pfrom/pto`） |
| `focusout` | `data-params`（`type/from/to/duration`） |
| `soundvolume` | `data-params`（`channel/volume/fadetime`） |
| `curtain` | `data-params`（`direction/fillfrom/fillto/fadetime`） |

注：裸赋值 `[delay=0.5]`（无括号）在解析时归一为 `delay` 类型，`data-cmd="Delay"`。

### 2.8 controller

```html
<div class="scene-entry scene-controller" data-type="controller" data-index="6"
     data-ctrlcmd="[Dialog]" data-cmd="Dialog">
</div>
```

| 属性 | 说明 |
|---|---|
| `data-ctrlcmd` | 原始控制命令 |
| `data-cmd` | 命令名（如 `Dialog` / `HEADER`） |

## 3. 音乐变化记录

`bgm-change` 独立于 scene-entry，位于文档尾部的 `bgm changes` 区块：

```html
<div class="bgm-change"
     data-line="12" data-type="PlayMusic"
     data-key="BGM_001" data-intro=""
     data-volume="1.0" data-fadetime="0"
     data-url="resource/audio/BGM_001.mp3">
</div>
```

| 属性 | 说明 |
|---|---|
| `data-line` | 原始脚本行号 |
| `data-type` | 命令类型（PlayMusic / PlaySound 等） |
| `data-key` | 循环音频 key |
| `data-intro` | 引入音频 key |
| `data-volume` | 音量（默认 1.0） |
| `data-fadetime` | 淡入淡出时长（默认 0） |
| `data-url` | 解析后的音频路径 |

## 4. 资源引用方式（阅读器识别）

阅读器在发送给渲染器前，将正文中的资源路径替换为本地可访问形式（如 Blob URL）。识别模式：

| 属性 / 标签 | 说明 |
|---|---|
| `data-url` | 图片 / 音频 / 任意资源 |
| `data-bg` | 背景图 |
| `data-image` | 角色立绘 |
| `data-loop` | 循环音频 |
| `src` | `<img>` / `<audio>` 等资源 |
| `href`（`.css`） | 样式表 |

已内联的协议（`https:` / `data:` / `blob:`）不处理。
