# SPAC3Y — Preview Page

A self-contained HTML/CSS/vanilla-JS preview of 3 sections. No build tools — every file
is pasteable directly into a Webflow **Embed** block.

## Files

| File | Purpose |
|------|---------|
| `index.html` | Combined preview — all 3 sections stacked. Open this to test in a browser. |
| `section-1-hero.html` | Hero scroll-scrub canvas. Self-contained. |
| `section-2-moodboard.html` | Static CSS-grid moodboard (5 images). No JS. |
| `section-3-card.html` | Glassmorphism AR/VR card with the hover "horror flicker". Self-contained. |
| `assets/` | All images + the extracted frame sequence. |

> **Serve over HTTP, not `file://`.** Section 1 loads the frame sequence with the Canvas
> API, which browsers block on the `file://` protocol. Locally: `python3 -m http.server`
> then open `http://localhost:8000/index.html`. In Webflow it's served over HTTP already.

## Section dividers

A light thin white line (`border-top: 1px solid rgba(255,255,255,.12)`) sits between every
block — hero↔moodboard and moodboard↔card. In the standalone snippets it lives on the top of
`section-2` and `section-3`, so the dividers appear automatically when the blocks are stacked
in Webflow.

## Frame sequence (Section 1)

- **Frames generated: 125** (`assets/frames/frame_001.webp` … `frame_125.webp`).
- Source `hero-video.mp4` is 1280×720, 10.0s, 24fps. Extracted at **~12.5fps** (auto-computed
  to land the count in the 120–130 range), scaled to a max width of 1920px (source is 1280px
  wide, so no upscaling), quality 75. More frames than the original 100 = a slightly smoother
  scrub on slow scrolls; the source has 240 frames total, so there's headroom for more if wanted.
- This ffmpeg build has no libwebp encoder, so frames were extracted as PNG via ffmpeg and
  converted to WebP with the standalone `cwebp -q 75`. To regenerate:

  ```bash
  # 125 frames from a 10s clip → ~12.5fps
  ffmpeg -i assets/hero-video.mp4 -vf "fps=12.5,scale='min(1920,iw)':-2:flags=lanczos" /tmp/f_%03d.png
  for f in /tmp/f_*.png; do cwebp -q 75 -m 4 "$f" -o "assets/frames/frame_$(basename "$f" .png | sed 's/f_//').webp"; done
  ```

  If you change the source clip length, keep frames in range by setting `fps ≈ 125 / duration_seconds`.
  **If the frame count changes, update `FRAME_COUNT` in the Section 1 script** (currently `125`,
  in both `index.html` and `section-1-hero.html`).

## Asset weight

| Group | Size |
|-------|------|
| Frame sequence (125 × WebP) | ~4.4 MB |
| Moodboard + card images (7 × JPG) | ~5.2 MB |
| `hero-video.mp4` (source only — not loaded by the page) | ~2.3 MB |
| **Total shipped to the browser (frames + JPGs)** | **~9.6 MB** |

The source JPGs are ~0.7–0.8 MB each straight from the originals. For production, run them
through an optimizer (or re-export as WebP) — the moodboard/card images are the biggest lever.

## Section-specific notes

- **Section 1:** pinned/sticky viewport with a `<canvas>`; scroll progress (0–1) maps to the
  frame index. All frames preload behind a `%` loading state before the scrub is meaningful,
  and updates run through `requestAnimationFrame` (not raw scroll events). The scroll distance
  is controlled by `.spacey-hero__track { height: 360vh }` — increase for a slower/smoother scrub.
- **Section 2:** intentionally minimal 3-column grid (`object-fit: cover`, `aspect-ratio: 4/3`,
  5th image spans 2 columns). Easy to rebuild later with native Webflow Image + Grid elements.
- **Section 3:** glassmorphism (`backdrop-filter: blur`) over a gradient backdrop so the blur
  has something to refract; inset white glow + scale-up (1.02) on hover. The flicker fires on
  `mouseenter`: 3–4 toggles between `card-ar-1`/`card-ar-2` with randomized 40–180ms gaps
  (`setTimeout` + `Math.random()`), settling on `card-ar-2`; `mouseleave` reverts instantly.
  `card-ar-2.jpg` is preloaded so the first flicker has no load flash.

## Asset name mapping

The originals were renamed into `assets/` for clean, snippet-friendly paths:

| Original | In `assets/` | Used by |
|----------|--------------|---------|
| `g1–g5.jpeg` | `moodboard-1…5.jpg` | Section 2 grid |
| `slide1.jpeg` | `card-ar-1.jpg` | Section 3 default image |
| `slide2.jpeg` | `card-ar-2.jpg` | Section 3 hover/flicker image |
| `hero-video.mp4` | `hero-video.mp4` | frame-sequence source (not loaded at runtime) |
| `g6.jpeg` | `moodboard-bg.jpg` | **unused** (was the wordmark-stripe background, now removed) |
