# Identity replacement workflow for the supplied video

This repository contains the source footage and reference photos. This guide is a reproducible workflow for making the requested identity replacement with a local video face-replacement tool. It does **not** alter the original media.

## Inputs

- Target video: `video_2026-09-27_20-09-41.mp4`
- Reference identity photos:
  - `photo_2026-09-27_20-08-11.jpg`
  - `photo_2026-09-27_20-08-19.jpg`
  - `photo_2026-09-27_20-08-24.jpg`
  - `photo_2026-09-27_20-08-30.jpg`
  - `photo_2026-09-27_20-08-37.jpg`
  - `photo_2026-09-27_20-08-43.jpg`
  - `photo_2026-09-27_20-08-54.jpg`
  - `photo_2026-09-27_20-09-01.jpg`
  - `photo_2026-09-27_20-09-20.jpg`
  - `photo_2026-09-27_20-09-25.jpg`

## Recommended processing

Use the official FaceFusion installation on a machine with a supported GPU if available. The command below uses reference-based selection so that only the intended main character is selected in the group scene. It enables only the face swapper: no expression restorer, frame enhancer, colorizer, crop, or output scaling is enabled, which best preserves the target video's crowd, actions, lighting, grain, and timing.

Run this from the repository root after installing FaceFusion:

```bash
#!/usr/bin/env bash
set -euo pipefail

sources=(
  photo_2026-09-27_20-08-11.jpg
  photo_2026-09-27_20-08-19.jpg
  photo_2026-09-27_20-08-24.jpg
  photo_2026-09-27_20-08-30.jpg
  photo_2026-09-27_20-08-37.jpg
  photo_2026-09-27_20-08-43.jpg
  photo_2026-09-27_20-08-54.jpg
  photo_2026-09-27_20-09-01.jpg
  photo_2026-09-27_20-09-20.jpg
  photo_2026-09-27_20-09-25.jpg
)

target=video_2026-09-27_20-09-41.mp4
output=video_2026-09-27_20-09-41_face-replaced.mp4

python facefusion.py headless-run \
  --source-paths "${sources[@]}" \
  --target-path "$target" \
  --output-path "$output" \
  --processors face_swapper \
  --face-selector-mode reference \
  --face-selector-order best-worst \
  --face-selector-gender male \
  --face-mask-types box occlusion \
  --face-mask-blur 0.20 \
  --output-video-encoder libx264 \
  --output-video-preset slow \
  --output-video-quality 95
```

The command deliberately does not set an output FPS, scale, crop, or `--skip-audio`; the target's vertical format, timing, and audio should therefore be retained. `box occlusion` masking is useful for the hand/smoking gesture because it lets foreground objects remain in front of the replacement face.

If the machine has CUDA, add this option; otherwise omit it or use the provider supported by the installation:

```bash
--execution-providers cuda
```

## Quality pass

1. Inspect a short section containing the clearest front-facing view before processing the full clip.
2. Check several turns, smoke/hand occlusions, and cuts for identity drift or mask edges.
3. If the face looks overly smooth, keep `face_enhancer` disabled. If edge blending is visible, adjust `--face-mask-blur` in small increments rather than changing the source video.
4. Confirm the output remains 9:16, has the same duration and frame rate as the target, and retains the original audio.
5. Keep the original video as the source of truth and export the result to a new filename.

## Important limitation

A true frame-consistent replacement cannot be rendered with the image-only tools available in this workspace. It requires a video-capable local or hosted runtime and model downloads. Use this workflow only with the subject's permission to use their likeness.

## References

- FaceFusion's official CLI documentation: <https://docs.facefusion.io/usage/cli-commands/general>
- Official path arguments: <https://docs.facefusion.io/usage/cli-arguments/paths>
- Official face selector arguments: <https://docs.facefusion.io/usage/cli-arguments/face-selector>
- Official processor arguments: <https://docs.facefusion.io/usage/cli-arguments/processors>
- Official mask arguments: <https://docs.facefusion.io/usage/cli-arguments/face-masker>
- Official output arguments: <https://docs.facefusion.io/usage/cli-arguments/output-creation>
