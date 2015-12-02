# bff 1.1 😘

bff (**B**yte **F**ile **F**ormat) is the format used by software to render or publish Bytes. A bff document contains plaintext JSON that describes all of the objects and instructions that make up a Byte.

##Overview

There are 3 top level properties in a bff document:

- `version` a string identifier that versions the document
- `background` an array that describes the background color of the Byte
- `objects` an array of objects that make up the Byte

##Rendering

The bff stage dimensions are always **324x570** points.

Objects in a bff are rendered from back-to-front, starting with the `background`, followed by the first item in the `objects` array, and so on.

Importantly, **the coordinate space in a bff is top-left aligned**.

```
(0, 0) -----------> (324, 0)
  |
  |
  |
  |
  |
  |
  V
(0, 570)
```
##Version

The format of the `version` property is `"1.2"`.

Versions are incremented with the following rules:

Given `"MAJOR.MINOR"`, we increment the
- MAJOR version when changes are made that are incompatible with with older renderers
- MINOR version when functionality is added in a backwards-compatible manner

🚨 **A bff must specify a version.**

##Background

The format of the `background` property is `[R, G, B, A]`, where valid color values are `0.0` to `1.0`. The property can also be two-dimensional (e.g. `[[R, G, B, A], [R, G, B, A]]`) to render a gradient background. If no background is specified, a default of `[1.0, 1.0, 1.0, 1.0]` is used.

##Objects

Objects make up most of a bff and are contained in the `objects` array.

All objects in a bff can have these properties:

name | description | type
--- | --- | --- | ---
`type` | **Required**. The type of object (e.g. text, video). | String
`frame` | **Required**. Position and size of the object.<br>Format is `[X, Y, WIDTH, HEIGHT]`. | Array of Floats
`name` | A unique name or ID for the object. Case sensitive. | String
`transform` | The top two rows of a 3x3 affine transformation matrix for rotation, scaling, translation. Applied **after** the `frame` is set.<br>Format is `[[A, B], [C, D], [TX, TY]]`. | Array of Arrays of Floats
`opacity` | The alpha value of the object.<br>Valid values are `0.0` to `1.0` | Float
`effects` | Animation effects that are applied to the object.<br>See [Effects](#effects) for possible values. | Array of Strings
`originalSrc` | Optional web url linking to the original source of the object. Example: http://cg-horses.tumblr.com/post/44175751464 | String

Object `type` values:
- [paragraph](#paragraph)
- [text](#text)
- [link](#link)
- [image](#image)
- [graphic](#graphic)
- [gif](#gif)
- [video](#video)
- [music](#music)

####Paragraph
**type:** `paragraph`

The Paragraph object renders text that flows and automatically wraps inside the `frame`. Paragraphs are best used to render text that is longer than 1-3 words.

name | description | type | default
:--- | :--- | :--- | :---
`text` | **Required**. The text to render. Cannot be purely whitespace. | String 
`color` | A color to render the text.<br> Format is `[R, G, B, A]`, where valid color values are `0.0` to `1.0`. | Array of Floats | `[0, 0, 0, 1]`
`style` | A style to render the text.<br>Valid values are `sans` and `serif`. | String | `sans`
`size` | Point size to render the text. Must be greater than 0. | Float | `17.0`
`alignment` | Text alignment to render the text.<br>Valid values are `left`, `center`, `right`. | String | `left`
`attributes` | Describes attributes to apply to the text.<br>Format is `[{"type": "bold", "range": [startIndex, length]}, {...}, {...}]`, where `type` can be `bold`, `italic`, or `bold-italic`. | Array of Dictionaries

####Text
**type:** `text`

Unlike the Paragraph object, **the Text object renders text that dynamically changes size** to fill its frame. Depending on their configuration, Text objects can either wrap based on rules set by the renderer, or not wrap at all. Text objects are always centered.

🚨 **Text objects are best used to render text between 1 and 3 words.**

name | description | type | default
:--- | :--- | :--- | :---
`text` | **Required**. The text to render. Cannot be purely whitespace. | String 
`color` | A color to render the text.<br> Format is `[R, G, B, A]`, where valid color values are `0.0` to `1.0`. | Array of Floats | `[0, 0, 0, 1]`
`style` | A style to render the text.<br>Valid values are listed under the [Text fonts](#text-1). | String | `sans`
`word-wrap` | If `auto`, wrapping will be determined by the client. If `manual`, text will only wrap where line breaks are encountered. It is preferable to specify a `manual` property and provide manual linebreaks so that wrapping is consistent across different renderers. | String | `auto`
`padding` | The padding (if any) to apply to the inside of the frame while dynamically sizing the text to fit. Must be greater than or equal to 0. | Float | `20.0`
~~`alignment`~~ | **Not supported**. Text objects are always centered.

####Link
**type:** `link`

Links are pressable objects that redirect the viewer to other Bytes or websites.

name | description | type | default
:--- | :--- | :--- | :---
`url` | **Required**. The URL to redirect to.<br>Both `byte://` and `http://` URL schemes are supported.<br>To redirect to a Byte by ID (rather than name), use `byte://byte.{BYTE_ID}`. | String
`title` | The title to display on the link. | String
`color` | A color to render the link.<br> Format is `[R, G, B, A]`, where valid color values are `0.0` to `1.0`. | Array of Floats | `[0, 0, 0, 1]`
`style` | A style to render the link.<br>Valid values are `sans` and `serif`. | String | `sans`
`description` | A description of the content behind the link. | String
~~`alignment`~~ | **Not supported**. Link titles are always centered.

####Image
**type:** `image`

Images are inline PNG, JPEG, or BMP files.

name | description | type | default
:--- | :--- | :--- | :---
`src` | **Required**. The URL for the image asset. Any URL on the web is valid. After a Byte is published, we may point to an internally cached version of the image in order to help performance. | String
`scaleMode` | A rule describing how the image should be resized inside its `frame`.<br>Valid values are `fit` and `fill`. | String | `fill`

####Graphic
**type:** `graphic`

Graphics are monochromatic PNG files that can dynamically change color.

name | description | type | default
:--- | :--- | :--- | :---
`src` | **Required**. The URL for the PNG file. Any URL on the web is valid. After a Byte is published, we may point to an internally cached version of the image in order to help performance. | String
`color` | A color to fill the graphic with.<br> Format is `[R, G, B, A]`, where valid color values are `0.0` to `1.0`. | Array of Floats | `[0, 0, 0, 1]`
~~`scaleMode`~~ | **Not supported**. Graphics always scale to fit to their `frame`.

####Gif
**type:** `gif`

![](http://i.giphy.com/sjuPRMd7ithh6.gif)

name | description | type | default
:--- | :--- | :--- | :---
`src` | **Required**. The URL for the GIF file. Any URL on the web is valid. After a Byte is published, we may point to an internally cached version of the GIF in order to help performance. | String
`scaleMode` | A rule describing how the GIF should be resized inside its `frame`.<br>Valid values are `fit` and `fill`. | String | `fit`

####Video
**type:** `video`

Videos are inline MP4 files. Videos **play automatically** and **loop on completion**. Videos should be limited to 10 seconds or less.

name | description | type | default
:--- | :--- | :--- | :---
`src` | **Required**. The URL to the MP4. Any URL on the web is valid. After a Byte is published, we may point to an internally cached version of the MP4 in order to help performance. | String
`muted` | Indicates whether or not the video should play sound | Boolean | `true`

####Music
**type:** `music`

Music is a MIDI-esque instruction set that is played by a built-in sequencer using the Byte soundfont. Music **plays automatically** and **loops on completion**. A speaker graphic is rendered at the object's `frame`.

🎹 **The Byte soundfont is included in this repository (`/Soundfont/Byte.sf`) for convenience.**

🚨 **A bff can contain only one music object.**

name | description | type | default
:--- | :--- | :--- | :---
`bpm` | **Required**. The sequences's beats-per-minute. Must be greater than 0. | Integer
`length` | **Required**. The length of the sequence, in bars.<br>Valid values are 2-16. Must be a multiple of 2.</br> | Integer
`instructions` | **Required**. An array of Music Instructions for the sequence. | Array of Music Instructions

#####Music Instructions

Music `instructions` are an array of **Beats**, which in turn are an array of **Hits**. The length of `instructions` (the number of **Beats**) must be at least `(length * 4)`.

###### Beats

Beats are arrays which contain a variable number of hits. The position of a beat inside the `instructions` array determines which point in the sequence's timeline the beat and its hits correspond to. All time is in 4:4. For example, a beat at `instructions[3]` represents the final beat of the first bar of the sequence. Beats themselves do not specify the time at which sound is played (see Hits below), but it is important that they are organized properly because they are used to buffer their corresponding hits.

###### Hits

Hits are the notes and drums inside a beat. Each hit is an object with the following properties:

name | description | type
:--- | :--- | :--- | :---
`time` | The time offset (in bars) in the sequence's timeline to play the hit. Can be subdivided to create half, quarter, 8th, 16th, or 32nd notes. Must be greater than or equal to 0. | Float
`type` | Specifies the type of hit, where `0 = instrument` and `1 = drum`. | Integer
`bank` | If type is `0`, specifies the type of instrument to play. If type is `1`, this value should always be `"drums"`. | String
`note` | If type is `0`, the value is in the format `"B/3"` and specifies the note and octave to play. To play a sharp note, a hash can be added (e.g. `"C#/3"`). Flat notes are not supported. Valid values are `A/-1` to `G/8`.<br><br>If the type is `1`, the value is in the format `"kick"` and specifies the type of drum to play. | String
`velo` | Specifies the velocity (typically volume) at which to play the hit, from `0` (none) to `127` (full). | Integer
`duration` | The duration (in bars) to play the note. Unsupported in current renderers and should be set to `1`. Must be greater than 0. | Float

**Supported bank names:**
`"bleep"`, `"meow"`, `"bass"`, `"ping"`, `"string"`, `"reso"`, `"arp"`, `"bark"`, `"mono1"`, `"mono2"`, `"mono3"`, `"funk"`, `"sax"`, `"bell"`, `"roboto"`, `"do"`

These banks correspond to sound banks `0` to `15` in the Byte Soundfont, except for `drums`, which corresponds to sound bank `127`.

**Supported drum hit names:**
`"Thump"`, `"Kick"`, `"Glitch"`, `"Snare"`, `"Clap"`, `"Tambourine"`, `"Whistle"`, `"Hat"`, `"Block"`, `"Stick"`, `"Shaker"`, `"Crash"`, `"Tom"`, `"Conga"`, `"Cowbell"`, `"Yeah"`

These hits correspond to MIDI notes `35` to `50` in the Byte Soundfont's `drums` bank.

##Effects
Effects are discrete animations that can be applied to an object and looped indefinitely. An object can have zero or more effects, and effects can be combined to create new effects (e.g. combining `"sin"` and `"cos"` creates a circular movement).

Effects that are currently supported are:

Animation `id` | | Description
:--------------|:-------|:-----------
`sin` | Sway vertically | Makes an object sway in a vertical pattern
`cos` | Sway horizontally | Makes an object sway in a horizontal pattern
`wave` | Wave | Makes an object wave left and right
`rotate` | Rotate clockwise | Makes an object rotate clockwise
`soon` | Soon Zoom | Zooms an object ominously
`fireworks` | Fireworks | Emits numerous copies of thing, like fireworks
`explode` | Explode | Forces the object to shatter into pieces
`pulse` | Pulse | Applies a subtle pulse to the object
`fade` | Fade | Fades the object in and out
`right-to-left` | Right to Left | Moves the object onscreen and offscreen from right to left

*Rules and pseudo-code to replicate each `effect` will be published soon.*

##Fonts & Typefaces
Rather than explicitly defining fonts, objects that render text contain a `style` property. It's up to each particular renderer to define which font it will use to interpret a given style.

Style | Font Guidelines
:-------------|:---------
`sans` | Sans-serif
`serif` | Serif
`mono` | Fixed-width, sans-serif, non-pixel
`eightbit` | Fixed-width, sans-serif, pixel
`punchout` | Inverted (letter are "punched out" of blocks), sans-serif
`cursive` | Script
`poster` | Sans-serif, heavy, tall/narrow
`tape` | All caps, sans-serif, wide/short
`book`| All caps, sans-serif, appears handmade or slightly retro

#### Royalty Free Font Recommendations
Below are some suggested royalty free fonts that work well in renderers. For convenience, the font files are also included in this repository (`/Fonts/*`).

Style | Info | URL | License
:---|:---|:---|:---
`sans` | **Aileron** | [URL](http://dotcolon.net/font/aileron/) | CC0 1.0 Universal (CC0 1.0)  Public Domain
`serif` | **Heurestica** | [URL](http://www.ctan.org/tex-archive/fonts/heuristica/) | SIL Open Font License (OFL)
`mono` | **Roboto Mono 400** | [URL](https://www.google.com/fonts/specimen/Roboto+Mono) | Apache License 2.0
`punchout` | **Labeler** | Custom font by Byte Inc. Included in this repo. | Public Domain
`eightbit` | **Pixel Grotesk** | Custom font by Byte Inc. Included in this repo. | Public Domain
`cursive` | **League Script** | [URL](https://www.theleagueofmoveabletype.com/league-script-number-one) | SIL Open Font License (OFL)
`poster` | **Pressuru** | [URL](http://abstrkt.ru/index.php?p=freefonts) | “You are totally free to use the typefaces listed below any way you like.”
`tape` | **Alfphabet IV** | [URL](http://openfontlibrary.org/en/font/alfphabet) | SIL Open Font License (OFL)
`book` | **st32k** | [URL](http://www.abstractfonts.com/font/15015) | SIL Open Font License (OFL)

#### Fonts used by Byte, Inc.
In case it's helpful, below are the fonts used in our official Byte renderers.

#####Paragraph
Ranges of paragraph text can be bold, italic, both, or regular. 

Style | Attribute | Font
:-------------|:---------------|:----------
`sans` | None | GraphikApp-Regular
`sans` | `italic` | GraphikApp-RegularItalic
`sans` | `bold` | GraphikApp-Semibold
`sans` | `bold-italic` | GraphikApp-SemiboldItalic
`serif` | None | Tiempos
`serif` | `italic` | Tiempos-Italic
`serif` | `bold` | TiemposSemibold
`serif` | `bold-italic` | TiemposSemibold-Italic

#####Text
Style | Font
:-------------|:---------
`sans` | GraphikApp-Semibold
`mono` | GT Cinetype
`punchout` | Labeler
`eightbit` | Pixel Grotesk
`cursive` | KGTheFighter
`poster` | Druk-Bold
`tape` | Tapeworm-Regular
`book`| St32k
`serif` | TiemposSemibold

#####Link
Style | Font
:-------------|:---------
`sans` | GraphikApp-Semibold
`serif` | TiemposSemibold


##Changelog

###Version 1.1
* Added optional `originalSrc` property to objects

###Version 1.2: 
* Added several new supported `effects` values
