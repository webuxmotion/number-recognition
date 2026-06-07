# CLAUDE.md — Numbers Recognition

Single file: `index.html` (~2060 lines). No build step, no dependencies.

## File structure

| Lines | Section |
|-------|---------|
| 1–395 | CSS |
| 396–430 | HTML skeleton |
| 431–494 | Canvas drawing (mouse + touch, rainbow brush) |
| 495 | `const SZ = 32` — normalize size |
| 496–594 | Algorithm: `toGrayscale`, `computeHistogram`, `otsuThreshold`, `binarize` |
| 595–700 | Algorithm: `boundingBox`, `bilinearResize`, `extractFeatures` |
| 701–900 | Template bank: `buildTemplates` (10 digits × 6 fonts × 2 weights = 120) |
| 901–943 | `classify`, `runPipeline`, `stat` helpers |
| 944–1230 | **I18N** — EN + UK translations; `setLang(lang)` |
| 1231–1440 | `STEP_CODE[]` — 8 source-code strings shown in the UI |
| 1441–1954 | `STEPS[]` — 8 step definitions (title, desc, render) |
| 1955–2054 | UI: `renderStep`, nav buttons, run button, event wiring |

## Algorithm pipeline (must stay in sync for templates and user input)

```
RGBA → grayscale → Otsu threshold → binarize → auto-invert
  → bounding box crop + 18% padding → bilinear resize 32×32
  → feature vector (1024 px + 32 row proj + 32 col proj = 1088 dims)
  → Euclidean nearest-neighbour against 120 templates
```

## Key conventions

- `hi(t)` / `hi2(t)` / `dim(t)` — inline HTML highlight helpers; used inside I18N `info()` functions and peekBox/infoBox calls.
- `infoBox(html)` — styled explanation block; always receives translated HTML from `I18N[currentLang].steps[i].info(data)`.
- `peekBox(title, rows)` — compact data preview; defined per step inside the `render()` function; NOT translated (values are numeric).
- `buildDataExpand(data, i)` — full expandable data inspector appended by `renderStep`, not inside individual `render()` calls.
- `codeExpand(title, src)` — syntax-highlighted code block; title comes from `I18N[currentLang].codeTitle`.
- `hlCode(raw)` — single-pass character tokenizer (not regex). Never pass already-emitted HTML through it again.

## I18N pattern

All user-visible text in step renders goes through `I18N[currentLang].steps[i].*`.  
Static DOM elements (title, buttons, hint) are updated by `setLang(lang)` via element IDs.  
`renderStep(i)` always re-renders from scratch using `currentLang` at call time.

Each step's I18N entry has:
- `title`, `desc` — used by `renderStep`
- `info(data)` — returns HTML string; may call `hi/hi2/dim`; may reference `SZ`
- Step-specific label helpers (e.g. `pixelLabel`, `croppedLabel`, `templateLabel`)

## Things to watch out for

- If you remove a variable from a `render()` function's infoBox block, check whether the **peekBox code below it** still references that variable — it won't have its own declaration.
- `STEPS[i].title` / `STEPS[i].desc` are stale (kept for dot tooltips fallback). Canonical text is in `I18N`.
- Template rendering uses an offscreen canvas — avoid touching `drawCanvas` during `buildTemplates`.
- `buildDataExpand` lazy-renders large arrays (280×280 = 78 400 values) on first `<details>` open via a `toggle` event — do not eagerly call the text builders.
