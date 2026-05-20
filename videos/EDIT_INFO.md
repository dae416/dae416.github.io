# Video Edit Info — Project Page

Generated: 2026-05-20  
Output directory: `cv/videos/`  
Target spec: **1920×1080 HD, 30fps, H.264, no audio, < 100 MB**

---

## 1. `squid_main.mp4`  — Squid-Inspired Propulsion

**Output:** 81 MB, 218.2s (3:38), 1920×1080, 30fps, ~2787 kbps

| # | Source file | Original spec | Clip in output | Playback | Notes |
|---|---|---|---|---|---|
| 1 | `squid/1_darpa_thrust_demo.mp4` | 2704×1520, **240fps**, 25.2s, 10.5MB | 0:00 – 0:25 | **1×** | High-speed camera footage. Scaled down to 1080p. At 240fps→30fps output, motion appears 8× slowed. Consider speeding up (`setpts=PTS/8`) if real-time feel desired. |
| 2 | `squid/2_SI_movie_v15_highres.mp4` | 1920×1080, 30fps, 193.0s, 421.7MB | 0:25 – 3:38 | **1×** | Supplementary movie (SI). Already 1080p — no scaling. Compressed 421MB → included in 81MB output via 3000kbps cap. |

### GPT 편집 프롬프트 (squid)
```
You are editing a science video for an academic project page.
The video "squid_main.mp4" is a concatenation of 2 clips (218s total):

Clip 1 (0:00–0:25): High-speed camera footage (originally 240fps, now at 30fps = 8× slow-motion).
  - Suggested edit: speed up to real-time (setpts=PTS/8) or keep slow-motion for visual impact.
  - Trim: keep only the most visually striking 5–8 seconds.

Clip 2 (0:25–3:38): SI supplementary movie (science animation/experiment).
  - Trim to the most informative 20–30 seconds.
  - Suggested sections to keep: [TODO: user should note key timestamps from original].

Final target: 20–40 seconds total, suitable as a muted autoplay background video for a research project card.
Output: 1920×1080, 30fps, H.264, no audio, < 10 MB for web.
```

---

## 2. `mudskipper_main.mp4`  — Bioinspired Locomotion

**Output:** 3.0 MB, 4.3s, 1920×1080, 30fps, ~5036 kbps

| # | Source file | Original spec | Clip in output | Playback | Notes |
|---|---|---|---|---|---|
| 1 | `mudskipper/1_mudskipper_46_60fps-4.gif` | 632×632 (square), ~12fps, 2.9s, 5.6MB | 0:00 – 0:03 | **1×** | Square GIF — padded to 16:9 with black bars (pillarbox). Upscaled from 632px to 1080p. |
| 2 | `mudskipper/2_mudskipper_cenimatic.gif` | 1242×698, ~12fps, 1.4s, 7.7MB | 0:03 – 0:04 | **1×** | Near-HD GIF — small upscale to 1920×1080. Cinematic shot. |

### GPT 편집 프롬프트 (mudskipper)
```
You are editing a short science video loop for an academic project page.
The video "mudskipper_main.mp4" is a 4.3-second concatenation of 2 GIF clips.

Clip 1 (0:00–0:03): Square footage (632×632) of mudskipper locomotion, padded to 16:9.
  - The black bars are distracting. Suggested: crop to 16:9 instead of pad (use crop=ih*16/9:ih).
  - Or: use a blurred/zoomed version of the same frame as background behind the clip.

Clip 2 (0:03–0:04): Cinematic mudskipper shot (1.4s).

Final target: loop seamlessly (~5–10s loop), 1920×1080, no audio.
Consider adding slow-motion effect (setpts=2*PTS) to extend each clip to ~6s each.
Remove black bars using crop filter instead of pad.
```

---

## ffmpeg 재편집 명령어 참고

### Clip 1 속도 조정 (squid, 8× 슬로우모션 → 실시간):
```bash
ffmpeg -i squid/1_darpa_thrust_demo.mp4 \
  -vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2,fps=30,setpts=PTS/8" \
  -t 3 clip1_realtime.mp4
```

### mudskipper 블랙바 제거 (crop 방식):
```bash
# 632×632 square → crop to 356×632 (16:9) then scale to 1920×1080
ffmpeg -ignore_loop 0 -i mudskipper/1_mudskipper_46_60fps-4.gif \
  -vf "crop=ih*16/9:ih,scale=1920:1080,fps=30,setpts=2*PTS" \
  -t 5.8 clip_mud1_cropped.mp4
```

### 전체 재컴파일 (편집 후):
```bash
ffmpeg -i clip1.mp4 -i clip2.mp4 ... \
  -filter_complex "[0:v][1:v]concat=n=2:v=1:a=0[v]" \
  -map "[v]" -c:v libx264 -b:v 2000k -pix_fmt yuv420p -an output.mp4
```
