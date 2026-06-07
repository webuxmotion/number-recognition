# Number Recognition

A single-file web app that recognises hand-drawn digits (0–9) using a pure JavaScript algorithm — no ML libraries, no network requests, works fully offline.

**Live demo:** [draw a digit and watch every step of the algorithm in real time]

---

## How it works

The app runs an 8-step pipeline entirely in the browser:

| Step | What happens |
|------|-------------|
| 1 | **Capture** — canvas is read into a flat RGBA pixel array |
| 2 | **Grayscale** — R/G/B collapsed to one brightness value using ITU-R BT.601 weights |
| 3 | **Otsu's threshold** — optimal binarization cut point found automatically from the histogram |
| 4 | **Binarize** — every pixel becomes exactly 0 (background) or 1 (ink); auto-invert if needed |
| 5 | **Bounding box** — tightest crop around ink pixels + 18% proportional padding |
| 6 | **Normalize** — bilinear resize to 32×32 |
| 7 | **Feature extraction** — 1024 pixel values + 32 row projections + 32 col projections = **1088-dim vector** |
| 8 | **Nearest neighbour** — Euclidean distance to 120 stored templates → confidence scores |

### Template bank

120 templates generated on page load (no network):  
**10 digits × 6 fonts × 2 weights** (Arial, Georgia, Courier New, Times New Roman, Verdana, Impact — regular + bold)

### Classification

For each of the 10 digits the minimum distance across all 12 font variants is taken.  
Confidence: `conf[i] = 1 / dist[i]`, then all 10 values are normalised to sum to 100%.

---

## Usage

Open `index.html` directly in any modern browser — no server, no install, no build step.

```
open index.html
```

Draw a digit on the canvas, hit **▶ Run Steps**, then step through the algorithm with **Prev / Play / Next**.  
Each step shows a visualisation, a plain-language explanation, the actual data values, and the source code that produced them.

---

## Features

- Rainbow brush (hue cycles 1.5°/pixel via HSL)
- EN / UK language toggle
- Expandable data panels showing full arrays (not samples)
- Syntax-highlighted source code per step
- Play mode auto-advances through all 8 steps

---

## Stack

Vanilla JS · Canvas API · Zero dependencies · ~2000 lines in one file

---

Made by [@webuxmotion](https://www.threads.com/@webuxmotion)
