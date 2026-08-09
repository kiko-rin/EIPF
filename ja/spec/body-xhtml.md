# body.xhtml コンテンツ形式 v2.0.1-beta1

> body.xhtml は Entry のコンテンツのデータ層であり、**線形の `scene-entry` 項目列**でコンテンツを保持します。

## 1. ドキュメントスケルトン

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="zh-CN">
<head>
  <meta charset="UTF-8"/>
  <title>第1章 開幕</title>
  <link rel="stylesheet" href="shared/scenario.css"/>
</head>
<body class="scenario-body">
  <!-- page: entry_001 -->
  <!-- title: 第1章 開幕 -->

  <!-- ═══ entries (index order) ═══ -->
  ...scene-entry 項目...

  <!-- ═══ bgm changes ═══ -->
  ...bgm-change 項目...
</body>
</html>
```

- 正しい XHTML 5、UTF-8、BOM 禁止。
- 各項目は `<div class="scene-entry ...">` ひとつ。
- リソースパスは EIPF ルートからの相対パス。
- 任意で `<script src="shared/scenario.js">` を注入可能。

## 2. 項目型（`data-type`）

本仕様は**汎用コンテンツ形式**を定義します。下表の「ソースコマンド」は意味の例示に過ぎません。各プロデューサは自作品のコマンド名をこれらの `data-type` にマッピングします。

| `data-type` | ソースコマンド（例） | 説明 |
|---|---|---|
| `dialog` | 発言者テキスト / プレーンテキスト | 発言者とテキストを含む会話 |
| `Image` | 背景 / 画像コマンド | 背景 / 画像 |
| `music` | 音楽コマンド | 音楽（イントロとループ） |
| `char` | キャラクターコマンド | 複数スロットを持つキャラクター |
| `charaction` | キャラクターアクションコマンド | アクション / 表情切替 |
| `theater` | theater | シネマ / ワイド字幕モード |
| `decision` | 選択コマンド | 選択分岐 |
| `sticker` | 字幕コマンド | 中央テキスト / 字幕 |
| `delay` | 遅延コマンド | ポーズ / 遅延 |
| `blocker` | オーバーレイコマンド | 画面フラッシュ白 / 黒 / 色マスク |
| `effect` | フィルターコマンド | カメラフィルター |
| `shake` | シェイクコマンド | カメラ振動 |
| `predicate` | 分岐述語コマンド | 選択分岐の述語 |
| `navigate` | ジャンプコマンド | ストーリージャンプ |
| `battle` | バトルコマンド | バトル開始 |
| `tutorial` | チュートリアルコマンド | チュートリアル起動 |
| `video` | ビデオコマンド | CG ビデオ |
| `curtain` | 転換コマンド | カーテン転換 |
| `controller` | その他の制御コマンド | ページング / メタ情報など |

> 各項目は `data-cmd`（コマンド名、例：`Blocker`）を持ち、レンダラーがソースコマンドを判別できます。新規実装型のパラメータは `data-params`（JSON）で完全に保持されます。

### 2.1 dialog

```html
<div class="scene-entry scene-dialogue" data-type="dialog" data-index="0"
     data-speaker="キャラA" data-thought="true">
  <span class="speaker">キャラA</span>
  <span class="text">（心の声）</span>
</div>
```

| 属性 | 説明 |
|---|---|
| `data-type` | `"dialog"` |
| `data-index` | インデックス（0 から増加） |
| `data-speaker` | 発言者（空 = ナレーション） |
| `data-thought` | 心の声フラグ |

### 2.2 Image

```html
<div class="scene-entry scene-image" data-type="Image" data-index="1"
     data-ctrlcmd="[Image(image=bg_01)]" data-url="resource/backgrounds/bg_01.png">
</div>
```

| 属性 | 説明 |
|---|---|
| `data-type` | `"Image"`（先頭大文字） |
| `data-ctrlcmd` | 元のコマンドテキスト |
| `data-url` | 背景 / 画像パス |

### 2.3 music

```html
<div class="scene-entry scene-music" data-type="music" data-index="2"
     data-ctrlcmd="[PlayMusic(key=..., intro=...)]"
     data-url="resource/audio/intro.mp3" data-loop="resource/audio/loop.mp3">
</div>
```

| 属性 | 説明 |
|---|---|
| `data-url` | イントロ音声パス |
| `data-loop` | ループ（key）音声パス |

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

- `data-url`（char 項目自身）は現在フォーカス中のキャラクターの立ち絵パス。
- 内部の 1 つ以上の `.char-slot` が具体的なキャラクターを記述します：

| 属性 | 説明 |
|---|---|
| `data-name` | 完全なキャラクター ID（`image_id + suffix`） |
| `data-image-id` | 画像 ID（`#suffix` より前の部分） |
| `data-suffix` | サフィックス（例：`#4`、なければ空） |
| `data-url` | 立ち絵パス |
| `data-x` | キャラクター中心 X（**キャンバス幅比 0~1**） |
| `data-h` | キャラクター高さ（**キャンバス高さ比 0~1**） |
| `data-w` | キャラクター幅（**キャンバス幅比 0~1**、任意） |

> **キャンバス規約**：プロデューサは固定キャンバス（例：1280×720 px）上で位置・サイズを算出し、**キャンバス比**に正規化します。リーダーは自身のビューポートに比例拡大して、どの解像度でも正しく表示します。
> スロット → `data-x` の値と意味はプロデューサが定義します（例：スロット番号でマッピング）。デフォルトは単一キャラ中央 `0.50`、`data-h` デフォルト `0.90`。

### 2.5 decision

```html
<div class="scene-entry scene-decision" data-type="decision" data-index="4"
     data-ctrlcmd="[Decision(options=...;..., values=...;...)]">
  <button class="choice-btn" data-choice-index="0">選択肢A</button>
  <button class="choice-btn" data-choice-index="1">選択肢B</button>
</div>
```

| 属性 | 説明 |
|---|---|
| `data-choice-index` | 選択肢インデックス |

注：生成されるボタンは `data-target` を**書きません**。分岐ジャンプはレンダラーが実装します（`predicate` と組み合わせ）。

### 2.6 sticker

```html
<div class="scene-entry scene-sticker" data-type="sticker" data-index="5"
     data-ctrlcmd="[Sticker(text=...)]" data-text="中央テキスト">
</div>
```

| 属性 | 説明 |
|---|---|
| `data-text` | 表示するテキスト |

### 2.7 delay / blocker / effect / shake / predicate / navigate / battle / tutorial / interlude / cgitem / focusout / soundvolume / curtain など

新規実装型はすべてのパラメータを `data-params`（JSON）で保持し、よく使う属性も併せて公開します：

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

| 型 | 主要属性 |
|---|---|
| `delay` | `data-time`（ミリ秒） |
| `blocker` | `data-params`（`a/r/g/b`、`fadetime`、`block`） |
| `effect` | `data-effect`（例：`Grayscale`） |
| `shake` | `data-params`（`duration/xstrength/ystrength/vibrato/randomness/fadeout`） |
| `predicate` | `data-references`（選択分岐の参照） |
| `navigate` | `data-cmd`（`SkipToThis` / `GotoPage`） |
| `battle` | `data-stage` |
| `tutorial` | `data-signal` |
| `interlude` | `data-params`（`channel/switch/maskid/size/offset` など） |
| `cgitem` / `hidecgitem` | `data-url` + `data-params`（`layer/sfrom/sto/sduration/afrom/ato/aduration/pfrom/pto`） |
| `focusout` | `data-params`（`type/from/to/duration`） |
| `soundvolume` | `data-params`（`channel/volume/fadetime`） |
| `curtain` | `data-params`（`direction/fillfrom/fillto/fadetime`） |

注：括弧なしの裸代入 `[delay=0.5]` は `delay` 型に正規化され、`data-cmd="Delay"` になります。

### 2.8 controller

```html
<div class="scene-entry scene-controller" data-type="controller" data-index="6"
     data-ctrlcmd="[Dialog]" data-cmd="Dialog">
</div>
```

| 属性 | 説明 |
|---|---|
| `data-ctrlcmd` | 元の制御コマンド |
| `data-cmd` | コマンド名（例：`Dialog` / `HEADER`） |

## 3. 音楽変化レコード

`bgm-change` は scene-entry から独立し、文書末尾の `bgm changes` セクションにあります：

```html
<div class="bgm-change"
     data-line="12" data-type="PlayMusic"
     data-key="BGM_001" data-intro=""
     data-volume="1.0" data-fadetime="0"
     data-url="resource/audio/BGM_001.mp3">
</div>
```

| 属性 | 説明 |
|---|---|
| `data-line` | 元スクリプトの行番号 |
| `data-type` | コマンド型（PlayMusic / PlaySound など） |
| `data-key` | ループ音声 key |
| `data-intro` | イントロ音声 key |
| `data-volume` | 音量（デフォルト 1.0） |
| `data-fadetime` | フェード時間（デフォルト 0） |
| `data-url` | 解決済み音声パス |

## 4. リソース参照パターン（リーダーが認識）

リーダーはレンダラーに渡す前に、リソースパスをローカルでアクセス可能な形式（例：Blob URL）に置き換えます。認識パターン：

| 属性 / タグ | 説明 |
|---|---|
| `data-url` | 画像 / 音声 / 任意リソース |
| `data-bg` | 背景画像 |
| `data-image` | キャラクター立ち絵 |
| `data-loop` | ループ音声 |
| `src` | `<img>` / `<audio>` など |
| `href`（`.css`） | スタイルシート |

すでにインライン化されたプロトコル（`https:` / `data:` / `blob:`）はそのままにします。
