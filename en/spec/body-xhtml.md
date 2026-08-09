# body.xhtml Content Format v2.0.1-beta1

> body.xhtml is the data layer of an Entry's content, carrying content as a **linear sequence of `scene-entry` items**.

## 1. Document Skeleton

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="zh-CN">
<head>
  <meta charset="UTF-8"/>
  <title>Chapter 1 Opening</title>
  <link rel="stylesheet" href="shared/scenario.css"/>
</head>
<body class="scenario-body">
  <!-- page: entry_001 -->
  <!-- title: Chapter 1 Opening -->

  <!-- ═══ entries (index order) ═══ -->
  ...scene-entry items...

  <!-- ═══ bgm changes ═══ -->
  ...bgm-change items...
</body>
</html>
```

- Valid XHTML 5, UTF-8, no BOM.
- Each item is a `<div class="scene-entry ...">`.
- Resource paths are relative to the EIPF root.
- `shared/scenario.js` may optionally be injected via `<script src="shared/scenario.js">`.

## 2. Item Types (`data-type`)

This specification defines a **generic content format**: the "source command" column below only illustrates semantics; producers map their own command names onto these `data-type` values.

| `data-type` | Source command (illustrative) | Description |
|---|---|---|
| `dialog` | speaker text / plain text | Dialogue with speaker and text |
| `Image` | background / image command | Background / image |
| `music` | music command | Music (intro and loop) |
| `char` | character command | Character with multiple slots |
| `charaction` | character action command | Character action / expression switch |
| `theater` | theater | Cinema / widescreen subtitle mode |
| `decision` | choice command | Choice branch |
| `sticker` | subtitle command | Centered text / subtitle |
| `delay` | delay command | Pause / delay |
| `blocker` | overlay command | Screen flash white / black / color mask |
| `effect` | filter command | Camera filter |
| `shake` | shake command | Camera shake |
| `predicate` | branch predicate command | Decision branch predicate |
| `navigate` | jump command | Story jump |
| `battle` | battle command | Enter battle |
| `tutorial` | tutorial command | Tutorial trigger |
| `video` | video command | CG video |
| `curtain` | transition command | Curtain transition |
| `controller` | other control commands | Pagination / metadata, etc. |

> Every item carries `data-cmd` (command name, e.g., `Blocker`) so the renderer can distinguish source commands; parameters of newly implemented types are fully preserved as `data-params` (JSON).

### 2.1 dialog

```html
<div class="scene-entry scene-dialogue" data-type="dialog" data-index="0"
     data-speaker="Character A" data-thought="true">
  <span class="speaker">Character A</span>
  <span class="text">(inner monologue)</span>
</div>
```

| Attribute | Description |
|---|---|
| `data-type` | `"dialog"` |
| `data-index` | Index (increments from 0) |
| `data-speaker` | Speaker (empty = narrator) |
| `data-thought` | Inner monologue flag |

### 2.2 Image

```html
<div class="scene-entry scene-image" data-type="Image" data-index="1"
     data-ctrlcmd="[Image(image=bg_01)]" data-url="resource/backgrounds/bg_01.png">
</div>
```

| Attribute | Description |
|---|---|
| `data-type` | `"Image"` (capital I) |
| `data-ctrlcmd` | Raw command text |
| `data-url` | Background / image path |

### 2.3 music

```html
<div class="scene-entry scene-music" data-type="music" data-index="2"
     data-ctrlcmd="[PlayMusic(key=..., intro=...)]"
     data-url="resource/audio/intro.mp3" data-loop="resource/audio/loop.mp3">
</div>
```

| Attribute | Description |
|---|---|
| `data-url` | Intro audio path |
| `data-loop` | Loop (key) audio path |

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

- `data-url` (of the char item itself) is the sprite path of the currently focused character.
- One or more `.char-slot` describe the specific characters:

| Attribute | Description |
|---|---|
| `data-name` | Full character ID (`image_id + suffix`) |
| `data-image-id` | Image ID (part before `#suffix`) |
| `data-suffix` | Suffix (e.g., `#4`, empty if none) |
| `data-url` | Sprite path |
| `data-x` | Character center X (**canvas-width ratio 0~1**) |
| `data-h` | Character height (**canvas-height ratio 0~1**) |
| `data-w` | Character width (**canvas-width ratio 0~1**, optional) |

> **Canvas convention**: producers compute character position/size on a fixed canvas (e.g., 1280×720 px) and normalize to **canvas ratios**. Readers scale proportionally to their own viewport for correct display at any resolution.
> Slot → `data-x` values and semantics are defined by the producer (e.g., mapped by slot index); default single-character center `0.50`, `data-h` default `0.90`.

### 2.5 decision

```html
<div class="scene-entry scene-decision" data-type="decision" data-index="4"
     data-ctrlcmd="[Decision(options=...;..., values=...;...)]">
  <button class="choice-btn" data-choice-index="0">Option A</button>
  <button class="choice-btn" data-choice-index="1">Option B</button>
</div>
```

| Attribute | Description |
|---|---|
| `data-choice-index` | Option index |

Note: produced buttons do **not** carry `data-target`; branch jumping is implemented by the renderer (in combination with `predicate`).

### 2.6 sticker

```html
<div class="scene-entry scene-sticker" data-type="sticker" data-index="5"
     data-ctrlcmd="[Sticker(text=...)]" data-text="centered text">
</div>
```

| Attribute | Description |
|---|---|
| `data-text` | Text to display |

### 2.7 delay / blocker / effect / shake / predicate / navigate / battle / tutorial / interlude / cgitem / focusout / soundvolume / curtain, etc.

Newly implemented types preserve all parameters as `data-params` (JSON) and expose common attributes:

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

| Type | Key attributes |
|---|---|
| `delay` | `data-time` (milliseconds) |
| `blocker` | `data-params` (`a/r/g/b`, `fadetime`, `block`) |
| `effect` | `data-effect` (e.g., `Grayscale`) |
| `shake` | `data-params` (`duration/xstrength/ystrength/vibrato/randomness/fadeout`) |
| `predicate` | `data-references` (decision branch references) |
| `navigate` | `data-cmd` (`SkipToThis` / `GotoPage`) |
| `battle` | `data-stage` |
| `tutorial` | `data-signal` |
| `interlude` | `data-params` (`channel/switch/maskid/size/offset`, etc.) |
| `cgitem` / `hidecgitem` | `data-url` + `data-params` (`layer/sfrom/sto/sduration/afrom/ato/aduration/pfrom/pto`) |
| `focusout` | `data-params` (`type/from/to/duration`) |
| `soundvolume` | `data-params` (`channel/volume/fadetime`) |
| `curtain` | `data-params` (`direction/fillfrom/fillto/fadetime`) |

Note: bare assignment `[delay=0.5]` (no parentheses) is normalized to the `delay` type with `data-cmd="Delay"`.

### 2.8 controller

```html
<div class="scene-entry scene-controller" data-type="controller" data-index="6"
     data-ctrlcmd="[Dialog]" data-cmd="Dialog">
</div>
```

| Attribute | Description |
|---|---|
| `data-ctrlcmd` | Raw control command |
| `data-cmd` | Command name (e.g., `Dialog` / `HEADER`) |

## 3. Music Change Records

`bgm-change` items are independent of scene-entry items and live in the trailing `bgm changes` section:

```html
<div class="bgm-change"
     data-line="12" data-type="PlayMusic"
     data-key="BGM_001" data-intro=""
     data-volume="1.0" data-fadetime="0"
     data-url="resource/audio/BGM_001.mp3">
</div>
```

| Attribute | Description |
|---|---|
| `data-line` | Source script line number |
| `data-type` | Command type (PlayMusic / PlaySound, etc.) |
| `data-key` | Loop audio key |
| `data-intro` | Intro audio key |
| `data-volume` | Volume (default 1.0) |
| `data-fadetime` | Fade duration (default 0) |
| `data-url` | Resolved audio path |

## 4. Resource Reference Patterns (recognized by readers)

Before handing content to the renderer, the reader replaces resource paths with locally accessible forms (e.g., Blob URLs). Recognized patterns:

| Attribute / tag | Description |
|---|---|
| `data-url` | Image / audio / any resource |
| `data-bg` | Background image |
| `data-image` | Character sprite |
| `data-loop` | Loop audio |
| `src` | `<img>` / `<audio>` etc. |
| `href` (`.css`) | Stylesheet |

Already-inlined protocols (`https:` / `data:` / `blob:`) are left untouched.
