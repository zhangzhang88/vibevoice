# Pi Agent 接管文档 — VibeVoice

更新时间：2026-10-05

## 1. 项目定位

本地路径：

```text
/Users/world/Downloads/code/vibevoice
```

这个项目最开始来自社区维护版 VibeVoice，现在同时作为用户本地的 AI 播客 / 多人对话语音生成工作区。

当前定位：

- `local-cosyvoice`：更适合单人旁白、个人声音。
- VibeVoice：主要用于长文本、多人播客、访谈式音频。

不要修改 `local-cosyvoice` 和 `my-voice-tts`。

用户偏好简单、按需生成，不希望常驻后台 API 服务。

## 2. Git / 上游状态

当前个人仓库：

```text
https://github.com/zhangzhang88/vibevoice
```

社区上游：

```text
https://github.com/vibevoice-community/VibeVoice.git
```

当前 remote 结构：

```text
origin   https://github.com/zhangzhang88/vibevoice.git
upstream https://github.com/vibevoice-community/VibeVoice.git
```

本次本地工作开始时对应的社区 commit：

```text
952326ddb264062466a888cf32a5b2f4e803e16e
Add info about transformers compatible checkpoints. (#67)
```

个人仓库是公开仓库，因此声音样本、头像、模型、生成音频和视频都不能上传。

## 3. 已验证机器与运行环境

机器：

- Mac mini M4
- 16GB 统一内存
- Apple Silicon
- macOS 27.0.1（build 26A434）

本地环境：

- Python 3.11.17
- `.venv/bin/python`
- 本地 uv：`.tools/uv-aarch64-apple-darwin/uv`
- MPS 可用
- 推理：MPS + FP16 + SDPA
- 系统没有 ffmpeg
- `.venv` 已安装 `imageio-ffmpeg==0.6.0`

注意：

- 当前 `.venv` 里没有 `python -m pip`。
- 如需安装包，可使用本地 uv。

## 4. 模型

本地模型：

```text
models/VibeVoice-1.5B
```

体积约 5.0GB，只保存在本机，已加入 `.gitignore`。

对当前 M4 16GB 来说，1.5B 是比较合适的版本。已经验证 MPS、FP16、长文本、多说话人、零样本声音克隆和中文播客生成。

中文生成偶尔会不稳定，标点和分段很重要。如果需要明显停顿，可以把同一个 Speaker 拆成两段连续输入。

## 5. 本地声音文件

只保留在本机：

```text
demo/voices/haoqin-25.mp3
demo/voices/zh-Haoqin_man.wav
demo/voices/老贺10秒.mp3
demo/voices/zh-Laohe_man.wav
demo/voices/hua.m4a
demo/voices/zh-Hua.wav
demo/voices/1.jpg
demo/voices/2.jpg
demo/voices/3.jpg
```

生产别名：

```text
Haoqin -> zh-Haoqin_man.wav
Laohe  -> zh-Laohe_man.wav
Hua    -> zh-Hua.wav
```

Haoqin 的 1.3x 加速实验已经因声音失真被用户否定并删除。除非用户明确要求重新实验，否则不要重新生成。

新增 `Hua` 声音：原始 `hua.m4a` 为约 13.38 秒、48kHz 立体声 AAC；已转换为 `zh-Hua.wav`（24kHz、单声道、PCM s16le），并使用 `seed 42` 完成 6.67 秒中文短句测试，生成成功。Hua 的头像为 `demo/voices/3.jpg`。

当前嘉宾：

```text
姓名：贺伟文
身份：东莞格拉美司董事长
```

## 6. 标准生成命令

```bash
.venv/bin/python demo/inference_from_file.py \
  --model_path models/VibeVoice-1.5B \
  --txt_path <script.txt> \
  --speaker_names Haoqin Laohe \
  --output_dir outputs \
  --device mps \
  --seed 42
```

当前完整播客第一次生成耗时约 17 分 35 秒，音频约 9 分 38 秒，RTF 约 1.83x。因此一句话有问题时，禁止整篇重生。

## 7. 当前播客

主题：

```text
地坪漆这个行业，现在到底还好不好做？
```

形式：

- 主持人：Haoqin
- 嘉宾：贺伟文
- 身份：东莞格拉美司董事长

本地完整脚本：

```text
tests/podcast-gelameisi-flooring-full.txt
```

已经完成的两处重要修正：

1. 开场从“地坪漆”改成“环氧地坪漆”，并在前面加入明显停顿。
2. 嘉宾介绍改成“今天请到的嘉宾，是东莞格拉美司的董事长贺总。”

## 8. 当前音频文件

原始完整版：

```text
outputs/podcast-gelameisi-flooring-full_generated.wav
```

第一次修复版：

```text
outputs/podcast-gelameisi-flooring-full-repaired-v1.wav
```

当前最新正式音频：

```text
outputs/podcast-gelameisi-flooring-full-repaired-v2.wav
```

当前时长：581.34 秒，约 9 分 41 秒。

修复 1：

```text
输入：tests/repair-opening-epoxy-flooring.txt
输出：outputs/repair-opening-epoxy-flooring_generated.wav
结果：在“环氧地坪漆”前形成约 0.64 秒停顿
```

修复 2：

```text
输入：tests/repair-guest-title-he-zong.txt
输出：outputs/repair-guest-title-he-zong_generated.wav
结果：“董事长”改为“董事长贺总”
```

## 9. 局部修复原则

以后如果用户指出某个时间点附近有问题，不要重生整篇。

正确流程：

1. 从**最新 repaired 完整 WAV** 开始。
2. 检查目标时间点前后的低能量 / 静音区。
3. 只在静音处切割，不能切到音节中间。
4. 只写一个很小的修复脚本。
5. 用同一个声音别名、同一个模型、同一个 `--seed 42` 重生。
6. 检查新片段时长和停顿。
7. 用 `soundfile + NumPy` 拼接。
8. 保存为 `repaired-v3.wav`、`repaired-v4.wav` 等新版本。
9. 不覆盖旧版。
10. 基于最新 WAV 重建视频。

如果只是停顿问题，优先直接插入静音。

## 10. 当前竖屏视频

当前用户确认通过的 **clean 模板参考基准**：

```text
outputs/podcast-video-background-husband-wife-vertical-clean-v2-1080x1920.png
outputs/podcast-husband-wife-video-channel-vertical-clean-v2.mp4
```

注意：这些文件属于 `outputs/`，只作为本机参考，不提交 Git。

固定参数：

- 1080×1920
- 9:16
- 30 fps
- H.264
- AAC 单声道
- 白色动态波形
- 波形带轻微发光
- 下方保留大面积字幕区

默认 clean 结构：

1. 顶部小标签
2. 主标题
3. 副标题
4. 两个圆形头像 + 角色/身份标签
5. 动态波形
6. 下方大块干净字幕区

默认禁止：

- 左右竖线
- 头像下面的横线
- 人物之间的分割线、圆点
- 引语、额外说明文字
- 字幕占位框
- “实时音频波形”
- “字幕安全区已预留”
- 其它让版面产生边框/框架感的装饰

公开画面只展示当前视频需要的角色/身份名称。内部声音别名（例如 `Haoqin`、`Hua`）只用于本地推理映射，不应该出现在发布画面中。

### 10.1 这次踩坑与防复发规则

曾经出现过两类错误：

1. 误把旧模板的竖线、横线、引语等重新带回，破坏 clean 结构。
2. FFmpeg 合成时把背景流错误地再次送入 overlay，导致标题、头像、文案在画面下半部又重复出现一套。

以后视频渲染必须遵守：

- 背景图只作为一次底图参与合成。
- waveform 需要 glow 时，先对 waveform `split`；一路做 `gblur` glow，一路保留白色 core，然后两路依次 overlay 到同一个背景上。
- 禁止重复 overlay 整个背景流。
- 不要直接渲染完整视频。先生成 3–8 秒 preview。
- preview 必须抽帧检查：只有一套标题/头像；动态波形可见；波形位置正确；字幕区没有重复图形；公开画面没有内部声音别名；分辨率为 1080×1920。
- preview 通过后再渲染完整视频。
- 完整视频完成后再次抽帧，并检查时长、编码、分辨率和音频流。

这套 preview → 抽帧验收 → 完整渲染 → 最终抽帧验收，是固定生产流程，不能省略。

## 11. 视频渲染

系统没有 ffmpeg。

获取本地 ffmpeg：

```bash
.venv/bin/python -c 'import imageio_ffmpeg; print(imageio_ffmpeg.get_ffmpeg_exe())'
```

如果 `imageio-ffmpeg` 缺失：

```bash
.tools/uv-aarch64-apple-darwin/uv pip install --python .venv/bin/python imageio-ffmpeg
```

## 12. 字幕状态

旧 SRT：

```text
outputs/podcast-gelameisi-flooring-full.srt
```

这个字幕文件已经过期，因为后面两次音频局部修复改变了整体时间轴。

正确顺序：

1. 先把所有音频问题修完
2. 锁定最终 WAV
3. 再生成 / 对齐最终字幕
4. 最后在剪映加字幕

## 13. Git / 隐私边界

不能提交：

- `models/`
- `outputs/`
- `.tools/`
- `.runtime/`
- `.venv/`
- 个人声音参考
- 头像
- `tests/*.txt` 本地播客脚本
- `voice-preview.html`

每次提交前：

```bash
git status --short
git diff --check
```

禁止随意 force push。

## 14. Pi 接管提示词

新开 Pi Agent 时直接说：

```text
请先读取 /Users/world/Downloads/code/vibevoice/AGENTS.md
和 /Users/world/Downloads/code/vibevoice/docs/PI-HANDOFF.md，
然后接管这个项目。先检查当前 Git 状态和本地最新音频/视频文件，
不要重新生成完整播客，也不要修改 local-cosyvoice 或 my-voice-tts。
```

这样即可继续，不需要重新回忆之前整段聊天。
