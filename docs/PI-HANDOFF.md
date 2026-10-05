# Pi Agent Handoff — VibeVoice

Updated: 2026-10-05

## Project

Local path:
    /Users/world/Downloads/code/vibevoice

Purpose:
- Keep the community VibeVoice code usable locally.
- Validate VibeVoice as a long-form multi-speaker podcast engine.
- Keep local-cosyvoice for single-narrator/personal-voice work.
- Prefer generate-on-demand; no permanent API daemon.

Community base when this work started:
    952326ddb264062466a888cf32a5b2f4e803e16e
    Add info about transformers compatible checkpoints. (#67)

Desired Git remotes:
    origin   https://github.com/zhangzhang88/vibevoice.git
    upstream https://github.com/vibevoice-community/VibeVoice.git

## Verified machine/runtime

- Mac mini M4
- 16 GB unified memory
- macOS 27.0.1 build 26A434
- Python 3.11.17
- project Python: .venv/bin/python
- local uv: .tools/uv-aarch64-apple-darwin/uv
- MPS available
- model inference: MPS + FP16 + SDPA
- no system ffmpeg
- imageio-ffmpeg 0.6.0 installed in .venv

Local model:
    models/VibeVoice-1.5B
Approximate size:
    5.0 GB

The 1.5B model is a practical fit for this 16 GB M4 machine.

## Local voices

Local-only files include:
    demo/voices/haoqin-25.mp3
    demo/voices/zh-Haoqin_man.wav
    demo/voices/zh-Haoqin_fast13_man.wav
    demo/voices/老贺10秒.mp3
    demo/voices/zh-Laohe_man.wav
    demo/voices/1.jpg
    demo/voices/2.jpg

Production aliases:
    Haoqin -> zh-Haoqin_man.wav
    Laohe  -> zh-Laohe_man.wav

Do not use zh-Haoqin_fast13_man.wav in production. The user rejected the 1.3x time-stretched voice because it sounded distorted.

Guest in the current episode:
- 贺伟文
- 东莞格拉美司董事长

## Canonical generation

    .venv/bin/python demo/inference_from_file.py \
      --model_path models/VibeVoice-1.5B \
      --txt_path <script.txt> \
      --speaker_names Haoqin Laohe \
      --output_dir outputs \
      --device mps \
      --seed 42

The first full episode took about 17m35s to generate roughly 9m38s of audio, so do not regenerate the whole episode for a single bad line.

## Current episode

Topic:
    地坪漆这个行业，现在到底还好不好做？

Local script:
    tests/podcast-gelameisi-flooring-full.txt

Important edits already made:
1. Opening changed from 地坪漆 to 环氧地坪漆 with a deliberate pause before the term.
2. Guest introduction changed to:
   今天请到的嘉宾，是东莞格拉美司的董事长贺总。

## Latest audio

Original:
    outputs/podcast-gelameisi-flooring-full_generated.wav

Repair v1:
    outputs/podcast-gelameisi-flooring-full-repaired-v1.wav

Current canonical audio:
    outputs/podcast-gelameisi-flooring-full-repaired-v2.wav

Current duration:
    581.34 seconds, about 9:41

Repair 1 input:
    tests/repair-opening-epoxy-flooring.txt
Repair 1 output:
    outputs/repair-opening-epoxy-flooring_generated.wav
Result:
    about 0.64 s pause before 环氧地坪漆

Repair 2 input:
    tests/repair-guest-title-he-zong.txt
Repair 2 output:
    outputs/repair-guest-title-he-zong_generated.wav
Result:
    董事长 -> 董事长贺总

## Local repair rule

Always start from the latest repaired full WAV.

For one bad sentence:
1. Find low-energy gaps around the reported timestamp.
2. Cut only on silence.
3. Generate only that sentence, or a small context window if needed.
4. Use the same voice alias and seed 42.
5. Splice with soundfile/NumPy.
6. Save as repaired-v3, repaired-v4, etc.
7. Keep all prior known-good versions.
8. Rebuild the latest video from the newest WAV.

For a pause-only problem, inserting silence directly may be preferable.

## Current vertical video

Background:
    outputs/podcast-video-background-gelameisi-vertical-clean-1080x1920.png

Current canonical video:
    outputs/podcast-gelameisi-video-channel-vertical-waveform-white-v2.mp4

Format:
- 1080x1920, 9:16
- H.264
- AAC mono
- 30 fps
- white waveform with slight glow
- clean lower area reserved for subtitles

Visual decisions:
- keep AI 播客 · 行业对谈 at top
- keep title and interview subtitle
- keep two avatars with names/roles
- no center divider
- no 实时音频波形 label
- no subtitle placeholder black box
- no 字幕安全区已预留 text
- bottom remains visually clean for captions

## Video rendering

No system ffmpeg is installed.

Find bundled ffmpeg:
    .venv/bin/python -c 'import imageio_ffmpeg; print(imageio_ffmpeg.get_ffmpeg_exe())'

If imageio-ffmpeg is missing locally:
    .tools/uv-aarch64-apple-darwin/uv pip install --python .venv/bin/python imageio-ffmpeg

## Subtitles

Existing old SRT:
    outputs/podcast-gelameisi-flooring-full.srt

Do not treat it as final. Later audio repairs changed timing.

Correct sequence:
1. finish all audio repairs
2. lock final WAV
3. generate/align final subtitles
4. add subtitles in 剪映

## Git/privacy boundary

Do not commit:
- models/
- outputs/
- .tools/
- .runtime/
- .venv/
- personal voice references
- avatars
- local podcast scripts under tests/*.txt
- voice-preview.html

## Pi startup prompt

The user can start Pi with:

请先读取 /Users/world/Downloads/code/vibevoice/AGENTS.md
和 /Users/world/Downloads/code/vibevoice/docs/PI-HANDOFF.md，
然后接管这个项目。先检查当前 Git 状态和本地最新音频/视频文件，
不要重新生成完整播客，也不要修改 local-cosyvoice 或 my-voice-tts。
