Try the tool here:

https://xcza.github.io/Color-Picker/

# Chromaxxing Color Picker

A compact, single-file color picker for **modern CSS color workflows**, perceptual editing, and wide-gamut color exploration.

Chromaxxing supports four modes:

- **OKLCH** — perceptual lightness, chroma, and hue editing
- **Vivid** — P3-aware vivid color tuning with Tone, Vividness, Peak Hue, and Hue
- **RGB** — direct channel editing and hex workflows
- **HSL** — familiar hue + picker-square interaction with CSS `hsl()` output

No build step. No framework. Just open the HTML file in a modern browser.

---

## Features

- Single-file HTML app
- Dark, compact UI
- Canvas-rendered sliders
- Editable numeric fields
- Editable color code and hex fields
- Alpha / opacity support
- Copy-from-swatch behavior
- Responsive layout
- OKLCH, RGB, HSL, and hex output
- Display-P3-aware previews when supported
- Computed vs fallback output for wide-gamut colors

---

## Running locally

Clone or download the repo, then open:

    index.html

For best results, use a browser with support for:

- `oklch(...)`
- `color(display-p3 ...)`
- canvas color space support

The picker still works without full wide-gamut support, but previews fall back to sRGB where needed.

---

## Project structure

    /
    ├─ index.html
    └─ README.md

The app currently keeps everything in `index.html`:

- markup
- styles
- color math
- canvas rendering
- input parsing
- drag/edit interactions

---

## Modes

## OKLCH

OKLCH mode is for direct perceptual editing.

Controls:

- **Lightness**
- **Chroma**
- **Hue**
- **Opacity**

Output example:

    oklch(0.6659 0.1564 86.84)
    oklch(0.6659 0.1564 86.84 / 74%)

OKLCH mode shows gamut boundaries where useful:

- **white marker** = sRGB boundary
- **black marker** = Display-P3 boundary

When a selected OKLCH color leaves Display-P3, the preview splits:

- **Computed** — the raw evaluated/clipped result
- **Fallback** — a P3-safe fallback that preserves lightness and hue while reducing chroma

The fallback row provides separate fallback code and hex values, which is useful for design tokens and browser QA.

---

## Vivid

Vivid mode is a guided, P3-aware editing model built on OKLCH.

Controls:

- **Tone** (`T`)
- **Vividness** (`V`)
- **Peak Hue** (`Hₚ`)
- **Hue** (`H`)
- **Opacity** (`A`)

Vivid mode still outputs standard CSS `oklch(...)`. It is an editing model, not a separate color space.

Vividness is normalized against the maximum P3 chroma available for the current Tone and Hue:

    chroma = vividness × maxChroma(tone, hue, Display-P3)

### Peak Hue

**Peak Hue** finds the most vivid P3-safe color for each hue by adjusting both Tone and Chroma.

Use it to answer:

> What is the strongest P3 color available at this hue?

### Hue

**Hue** changes hue while keeping Tone and Vividness fixed.

This rotates the color identity while preserving the same overall tone/vividness relationship.

### Active hue control

Vivid mode has two hue behaviors:

- **Peak Hue**
- **Hue**

Only one is active at a time. The active behavior appears as the vertical hue slider. The inactive horizontal control collapses into a compact animated switch row.

---

## RGB

RGB mode is for direct channel editing.

Controls:

- **Red**
- **Green**
- **Blue**
- **Opacity**

Output example:

    rgb(118 0 255)
    rgb(118 0 255 / 0.7)

RGB also supports direct hex editing.

---

## HSL

HSL mode uses a familiar app-style picker layout.

- Preview becomes smaller
- A square color field appears
- The square uses HSV-style interaction
- Output remains CSS `hsl(...)`

Output example:

    hsl(270 100% 50%)
    hsl(270 100% 50% / 0.7)

---

## Editing

Most displayed values can be edited inline:

- slider values
- color code
- hex
- fallback code
- fallback hex
- opacity

Supported inputs include:

- `#7600ff`
- `oklch(0.3657 0.2375 271.03)`
- `rgb(118 0 255 / 0.7)`
- `hsl(270 100% 50% / 0.7)`

---

## Copy behavior

Click the preview swatch to copy the current color code.

If the preview is split:

- click **Computed** to copy computed code
- click **Fallback** to copy fallback code

---

## Design philosophy

Chromaxxing is opinionated.

It is built to make color-space behavior visible instead of hiding it.

Core ideas:

- preserve the selected color state
- separate selected, computed, and fallback colors
- show gamut limits clearly
- keep controls honest about what they adjust
- make OKLCH and wide-gamut color easier to explore

---

## Good use cases

- exploring OKLCH
- building CSS color tokens
- testing sRGB and P3 behavior
- finding vivid P3-safe colors
- comparing computed vs fallback colors
- generating OKLCH, RGB, HSL, and hex values

---

## Future ideas

- saved swatches
- palette export
- URL serialization
- keyboard nudging
- token export
- contrast checking
- explicit P3 CSS export

---

Built as an experimental color picker focused on **OKLCH**, **Vivid P3 color exploration**, **gamut-aware UI**, and modern CSS color output.
