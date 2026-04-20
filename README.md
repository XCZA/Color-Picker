Try the tool here:

https://xcza.github.io/Color-Picker/


# Chroma Colo-Picker

A compact, single-file color picker focused on **modern CSS color workflows**.

Chroma supports three working modes:

- **OKLCH** for perceptual color work and gamut exploration
- **RGB** for direct channel editing and hex-oriented workflows
- **HSL** for the familiar app-style hue + square picker UI, with CSS-safe `hsl()` / `hsla()` output

The project is intentionally lightweight: no build step, no framework, no external app shell. Open the HTML file in a modern browser and it runs.

---

## Why this picker exists

Most color pickers are designed around sRGB and hide what happens when a color leaves a display gamut.

This picker takes a different approach:

- it lets you work directly in **OKLCH**
- it shows where colors are valid in **sRGB** and **Display-P3**
- it separates the idea of a **computed color** from a **perceptual fallback**
- it keeps the UI focused on what each control is actually adjusting, instead of always previewing the final clipped result

That makes it useful both as a practical picker and as a learning tool.

---

## Features

### General

- Single-file HTML app
- Dark UI with canvas-rendered sliders
- Editable numeric fields
- Editable color code and hex fields
- Copy-from-swatch behavior
- Alpha support across all modes

### Color spaces

- **OKLCH** output as CSS `oklch(...)`
- **RGB** output as CSS `rgb(...)`
- **HSL** output as CSS `hsl(...)` / `hsla(...)`
- Hex display for current computed color

### Wide-gamut support

- Detects and uses **Display-P3** when the browser supports it
- Distinguishes **sRGB** and **P3** boundaries in OKLCH mode
- Shows a **P3** badge when a color is inside P3 but outside sRGB

---

## Running the project

There is no install process.

1. Clone or download the repository
2. Open the main HTML file in a modern browser

For example:

```bash
open "index.html"
```

or just double-click the file.

For best results, use a browser with support for:

- `oklch(...)`
- `color(display-p3 ...)`
- canvas color space support

The picker still works without full wide-gamut support, but wide-gamut previewing will fall back to sRGB.

---

## Project structure

This project is designed to stay simple.

```text
/
├─ index.html   # app UI, styles, state, rendering, color math
└─ README.md
```

The current implementation keeps everything in one file:

- UI markup
- CSS
- color conversion math
- canvas rendering
- input parsing
- interaction logic

That makes it easy to tweak quickly and easy to host anywhere.

---

## Modes

## OKLCH mode

This is the most distinctive part of the picker.

OKLCH mode is built around three ideas:

1. **Perceptual editing** — you edit lightness, chroma, and hue directly
2. **Gamut awareness** — the UI shows where a color is valid in sRGB and P3
3. **Computed vs fallback output** — when a color leaves P3, the picker shows both the raw computed result and a perceptual fallback

### OKLCH output

The main code field uses standard CSS syntax:

```css
oklch(0.6659 0.1564 86.84)
oklch(0.6659 0.1564 86.84 / 0.74)
```

### What “computed” means

In this picker, **computed** means:

> “What RGB color do you get if you directly evaluate the current OKLCH coordinates?”

If the selected OKLCH color falls outside the target gamut, the math can still produce an RGB result after clamping. That result may shift hue or appearance.

Example:

- selected color: a high-chroma yellow in OKLCH
- computed result: may turn more orange after channel clipping

This picker keeps showing that computed result instead of hiding it.

### What “fallback” means

The **fallback** is the nearest visually sensible color used by the picker when the selected OKLCH value leaves **Display-P3**.

In the current implementation, the fallback is built by:

- keeping the current **lightness** and **hue**
- reducing **chroma** until the color fits inside **P3**
- converting that fallback RGB back to an OKLCH code for display in the fallback row

That means the fallback row is not just “another label” for the same color. It is a separate, accessible code path designed to preserve the original hue intent as much as possible.

### Split swatch behavior

When the selected OKLCH color is inside P3, the preview shows a normal single swatch.

When the selected OKLCH color goes outside P3, the preview splits into two halves:

- **Computed** — the raw computed result
- **Fallback** — the last valid P3 color

This makes the tradeoff visible without forcing the picker state to jump.

### Why the fallback row exists

When the swatch splits, the values area also adds:

- **Color Code (Fallback)**
- **Hex (Fallback)**

This gives you two usable outputs:

- the code you actually selected
- the display-safe fallback you may want to ship

That is especially helpful for design systems, design tokens, and browser QA.

### Gamut markers

OKLCH sliders include gamut markers so you can see where values stop being valid.

- **white line** = sRGB boundary
- **black line** = Display-P3 boundary

These markers appear only on sliders where they help explain the range. The vertical hue slider intentionally hides them to keep the hue strip visually clean.

### Vertical slider behavior in OKLCH mode

The vertical sliders are intentionally not all rendered the same way.

#### Lightness slider

The vertical **L** slider stays a simple **black → white** gradient.

It does **not** recolor itself based on the current hue/chroma, because its purpose is to communicate the **lightness axis** itself. Boundary markers are still drawn over it.

#### Chroma slider

The vertical **C** slider shows the active hue across chroma levels.

Above the valid P3 boundary, the slider does **not** wander into clipped hues. Instead, it holds the **last valid color**. This keeps the chroma track visually tied to one hue instead of letting clipping turn it into an unrelated color.

#### Hue slider

The vertical **H** slider behaves differently from the chroma slider.

It always shows a full hue spectrum, even when some hues are out of gamut. Rather than breaking into gaps, it fills those regions with fallback colors so the strip stays readable and continuous.

Also, it does **not** update from current lightness the same way the contextual sliders do. Its job is to communicate the hue dimension itself.

### Horizontal slider behavior in OKLCH mode

The horizontal sliders are more contextual.

They show how the current color changes as you adjust the active value, including the places where the color leaves a gamut. This makes them useful for precision tuning.

### Out-of-gamut state handling

A key design goal is that the picker **does not collapse your selection** just because it is out of gamut.

That means:

- the OKLCH sliders keep the original selected values
- switching to RGB shows the computed RGB color
- switching back to OKLCH restores the raw selected OKLCH color, unless you actually edited RGB

This is important, because otherwise you lose the distinction between:

- the color you selected
- the color the display can currently show

---

## RGB mode

RGB mode is straightforward by design.

- edit **R**, **G**, and **B** channels directly
- see live `rgb(...)` output
- edit hex directly
- alpha stays available as a separate control

RGB is also the interchange mode for parsed hex and functional RGB input.

---

## HSL mode

HSL mode uses a more familiar picker layout.

### Layout changes in HSL mode

- the left preview becomes a **half-width output swatch**
- the freed space becomes a **square color field**
- the square behaves like the common app picker:
  - top-left = white
  - top-right = full hue color
  - bottom = black
- only one vertical slider remains: **Hue**
- alpha moves to the **third horizontal slider**

### Why the square behaves like HSV

Although the output format is HSL, the square uses an **HSV-style interaction model** because that is what most people expect visually from this kind of picker.

So in HSL mode:

- the square is used for intuitive picking
- the final output is still formatted as CSS-safe `hsl(...)` / `hsla(...)`

Example output:

```css
hsl(50 100% 50%)
hsla(50 100% 50% / 0.74)
```

---

## Editing and interaction model

### Dragging

- drag sliders for continuous updates
- drag the HSL square to pick saturation/value visually

### Inline editing

Most displayed values can be edited inline:

- channel values
- color code
- hex
- fallback code
- fallback hex
- alpha

Supported input types include:

- hex: `#7600ff`
- OKLCH: `oklch(0.3657 0.2375 271.03)`
- RGB: `rgb(118 0 255 / 0.7)`
- HSL: `hsl(270 100% 50% / 0.7)`

### Copy behavior

Copying is attached to the preview swatch rather than the value chips.

- single swatch: click to copy the main color code
- split swatch: click the **computed** half to copy computed code, or the **fallback** half to copy fallback code

---

## Browser support notes

The picker tries to use the most capable rendering path available.

- If the browser supports `color(display-p3 ...)`, P3 previews are shown directly.
- If the browser supports `oklch(...)`, the preview can use native OKLCH CSS output.
- Otherwise, colors fall back to RGB output.

So the UI is progressive:

- modern browsers get the richer version
- older browsers still get a usable picker

---

## Design philosophy

This picker is opinionated.

It is not trying to hide color-space complexity. It is trying to make that complexity visible and useful.

The main ideas are:

- preserve the **selected color state**
- separate **selection**, **computation**, and **fallback**
- show gamut boundaries instead of pretending they do not exist
- keep controls focused on the value space they represent
- make wide-gamut behavior understandable at a glance

---

## Good use cases

- exploring OKLCH for design systems
- checking whether a color survives sRGB fallback
- building CSS color tokens
- comparing computed vs fallback output
- teaching the difference between perceptual and clipped color results
- generating HSL, RGB, and OKLCH values from one UI

---

## Future ideas

A few natural next steps for the project:

- export palettes or saved swatches
- add named preset colors
- add URL serialization for shareable states
- add keyboard nudging for all major controls
- add side-by-side token export for multiple formats
- add explicit P3 CSS export when useful

---

## License

Add the license you want to use here.

For example:

- MIT
- Apache-2.0
- GPL-3.0

---

## Credits

Built as an experimental color picker focused on **OKLCH**, **gamut-aware UI**, and modern CSS color output.
