# AGENTS.md

## 项目用途

这是用户在本机使用的 VibeVoice 播客工作区，当前已在 **Mac mini M4 / 16GB 统一内存** 上完成验证。

开始较大的任务前，先阅读：

1. `README.md`
2. `docs/PI-HANDOFF.md`

## Agent 工作规则

- 不要修改兄弟项目 `local-cosyvoice` 或 `my-voice-tts`。
- 除非用户明确要求，否则不要把本项目做成常驻 API 服务。
- 不要提交模型权重、本地运行环境、生成后的音频/视频、个人声音参考、头像、本地播客脚本。
- 长播客出现一句错误时，不要整篇重新生成；优先做**局部重生 + 音频拼接**。
- 已经可用的 WAV / MP4 不要覆盖，继续使用 `v2`、`v3` 这类版本号保存。
- 中文生成有时不稳定。需要明显停顿时，可以把同一个 Speaker 拆成连续两段。
- 当前生产流程固定使用 `--seed 42`。
- 只要音频时长发生变化，旧字幕时间轴就视为失效。字幕应在最终音频锁定后再统一生成。

## 当前本机环境

项目路径：

```text
/Users/world/Downloads/code/vibevoice
```

已验证环境：

- macOS 27.0.1（build 26A434）
- Apple M4
- 16GB 统一内存
- Python 3.11.17
- MPS 可用
- 本地模型：`models/VibeVoice-1.5B`
- 推理方式：MPS + FP16 + SDPA
- 系统没有安装 ffmpeg
- `.venv` 已安装 `imageio-ffmpeg`，用于本地渲染 MP4

## 本地声音别名

```text
Haoqin -> demo/voices/zh-Haoqin_man.wav
Laohe  -> demo/voices/zh-Laohe_man.wav
```

不要把 `demo/voices/zh-Haoqin_fast13_man.wav` 用于正式生成。这个 1.3x 加速实验已经被用户否定，因为声音会失真。

## 标准推理命令

```bash
.venv/bin/python demo/inference_from_file.py \
  --model_path models/VibeVoice-1.5B \
  --txt_path <script.txt> \
  --speaker_names Haoqin Laohe \
  --output_dir outputs \
  --device mps \
  --seed 42
```

## 局部修复工作流

当用户指出某个时间点附近音频有问题时：

1. 检查该时间点前后的一小段音频。
2. 找自然静音 / 低能量位置作为拼接边界。
3. 只重生错误的一句，必要时带上前后少量上下文。
4. 把新片段拼回新的完整 WAV。
5. 保留上一版完整 WAV，不覆盖。
6. 如果已有视频，就基于最新 WAV 重新生成视频。
7. 所有音频问题修完后，再统一生成最终字幕。

如果只是停顿不对、文字本身没错，优先考虑直接插入静音，而不是重生语音。

## 当前视频规范

- 1080×1920
- 9:16 竖屏
- 用于微信视频号
- 白色动态波形
- 波形带轻微发光
- 下方保留大块干净区域给字幕
- 不显示“实时音频波形”
- 不显示字幕占位黑框
- 不显示“字幕安全区已预留”
- 两个头像之间不放分割线

## Git 规则

预期 remote：

```text
origin   https://github.com/zhangzhang88/vibevoice.git
upstream https://github.com/vibevoice-community/VibeVoice.git
```

每次 push 前先检查：

```bash
git status --short
git diff --check
```

除非用户明确要求，否则禁止 force push。
