# AGENTS.md

## Purpose

This fork is the user's local VibeVoice podcast workspace on a Mac mini M4 with 16 GB unified memory.

Before substantial work, read:
1. README.md
2. docs/PI-HANDOFF.md

## Guardrails

- Do not modify sibling projects local-cosyvoice or my-voice-tts.
- Do not create an always-on service unless explicitly requested.
- Do not commit model weights, local runtimes, generated media, personal voice references, avatars, or local podcast scripts.
- Do not regenerate a full long-form podcast just to fix one bad sentence. Prefer segment regeneration and splicing.
- Keep previous good WAV/MP4 versions; create v2, v3, etc.
- Chinese generation can be unstable. For a stronger pause, split text into consecutive segments with the same speaker.
- Current production seed is 42.
- After any audio timing change, older subtitles are stale. Generate final subtitles only after audio is locked.

## Current local environment

Project:
    /Users/world/Downloads/code/vibevoice

Validated runtime:
- macOS 27.0.1 build 26A434
- Apple M4, 16 GB unified memory
- Python 3.11.17
- MPS available
- VibeVoice 1.5B at models/VibeVoice-1.5B
- inference: MPS + FP16 + SDPA
- no system ffmpeg
- imageio-ffmpeg is installed in .venv for local MP4 rendering

Voice aliases:
- Haoqin -> demo/voices/zh-Haoqin_man.wav
- Laohe -> demo/voices/zh-Laohe_man.wav

Do not use demo/voices/zh-Haoqin_fast13_man.wav for production; the 1.3x stretch was rejected because it distorted the voice.

## Canonical inference command

    .venv/bin/python demo/inference_from_file.py \
      --model_path models/VibeVoice-1.5B \
      --txt_path <script.txt> \
      --speaker_names Haoqin Laohe \
      --output_dir outputs \
      --device mps \
      --seed 42

## Repair workflow

When a timestamp is reported as wrong:
1. Inspect a small window around the timestamp.
2. Find silent/low-energy splice boundaries.
3. Regenerate only the bad sentence or a short context window.
4. Splice into a new full WAV.
5. Keep the previous WAV unchanged.
6. Rebuild the video from the newest WAV.
7. Regenerate subtitles only after all audio fixes are finished.

If only a pause is wrong and the words are correct, inserting silence may be better than regenerating speech.

## Current video format

- 1080x1920
- 9:16 vertical for WeChat Video Channels
- bright white animated waveform
- clean lower area for subtitles
- no waveform label
- no subtitle placeholder box/text
- no divider between avatars

## Git policy

Expected remotes:
    origin   https://github.com/zhangzhang88/vibevoice.git
    upstream https://github.com/vibevoice-community/VibeVoice.git

Before push:
    git status --short
    git diff --check

Never force-push unless explicitly requested.
