# CV Website (dae416.github.io)

Local repo: this directory  
Live: https://dae416.github.io

## Thumbnail rule — `thumbs/` folder

All thumbnail assets used in `publications.html` (and any other page thumbnail) must live in `thumbs/` at the repo root. **Never reference originals directly in HTML for thumbnails.**

### When asked to add or replace a thumbnail

**Video → `thumbs/<filename>.mp4`**

```
TARGET_KB=970
TRIM=$(python3 -c "d=$(ffprobe -v quiet -show_entries format=duration -of default=noprint_wrappers=1:nokey=0 INPUT.mp4 | grep -oE '[0-9.]+' | head -1); print(min(d, 8.0))")
BITRATE=$(python3 -c "print(min(int(970*8/$TRIM), 1400))")
ffmpeg -y -i INPUT.mp4 -t $TRIM -vf scale=480:-2 \
  -c:v libx264 -b:v ${BITRATE}k -maxrate ${BITRATE}k -bufsize $((BITRATE*2))k \
  -movflags +faststart -an thumbs/OUTPUT.mp4
```

Rules:
- Scale: 480px wide, height auto (`scale=480:-2`)
- Trim: max 8 seconds
- Target: ~1 MB (bitrate = min(970×8÷duration, 1400) kbps)
- No audio (`-an`)
- `+faststart` for web streaming

**Image → `thumbs/<filename>.png`**

```
ffmpeg -y -i INPUT.png -vf scale=480:-2 thumbs/OUTPUT.png
```

- Scale to 480px wide, keep PNG format
- Images already under ~300 KB: just copy to `thumbs/`

### HTML reference

Always use `src="thumbs/<filename>"` in HTML — never the original path.

### Folder layout

```
cv/
  thumbs/          ← all thumbnail-sized assets (≤1MB each)
  videos/          ← originals (full resolution, not linked in HTML)
  images/          ← originals (not linked in HTML for thumbnails)
  publications.html
  ...
```

## Publications page — special cases

- `ICB_combined.mp4` has JS zoom effect: `transform-origin` switches at 3.92s. The thumbnail version is trimmed to 8s so the 3.92s mark is preserved — do not change this value.
- `pulsed_jet_optimization.mp4` uses `data-rate="2"` (2× playback speed via JS).

## General site notes

- CSS: `assets/css/daehyun2.css` (rename file for cache busting on major changes)
- Gallery carousels: max 3 visible slides, never use `dc-gallery-fullwidth`
- All `<video>` elements: `muted loop playsinline preload="metadata"`, autoplay via IntersectionObserver
- Google Drive file paths require `find | while read` + copy to `/tmp` before ffmpeg processing
