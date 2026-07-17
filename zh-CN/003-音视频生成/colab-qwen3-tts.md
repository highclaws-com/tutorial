# 在 Google Colab 跑通 Qwen3-TTS 1.7B CustomVoice

## 目标

在免费 Google Colab 的 T4 GPU 上运行：

- 模型：`Qwen/Qwen3-TTS-12Hz-1.7B-CustomVoice`
- 英文声音：`Ryan`
- 产物：`qwen3_tts_ryan.wav`

本次验证成功：模型在 Colab T4 上完成推理，并生成了约 229 KB（234,284 bytes）的 WAV 音频。

---

## 推荐工作流：本地生成 Notebook，再上传

不要在 Colab 的网页编辑器中逐行手输大段代码。

1. 在本地生成并检查 `.ipynb` 文件（合法 JSON）。
2. 打开 Colab，使用 **File → Upload notebook** 上传。
3. 确认页面标题变成上传的 notebook 名称，并显示已保存。
4. 使用 **Runtime → Change runtime type → T4 GPU**。
5. 运行单元格。

本次 notebook 名称：`Qwen3-TTS-1.7B-CustomVoice.ipynb`。

---

## Colab 安装与 GPU 验证

```python
!nvidia-smi
import sys
!{sys.executable} -m pip install -q -U qwen-tts soundfile
```

确认顶部状态栏显示 **T4 (Python 3)**，并且 `nvidia-smi` 有 GPU 输出。

---

## 加载模型

在免费 T4 上，为避免安装/编译 FlashAttention，使用 PyTorch SDPA：

```python
import torch
from qwen_tts import Qwen3TTSModel

assert torch.cuda.is_available(), "No GPU detected. Select Runtime → Change runtime type → GPU, then reconnect."
print("GPU:", torch.cuda.get_device_name(0))
print("CUDA:", torch.version.cuda)

model = Qwen3TTSModel.from_pretrained(
    "Qwen/Qwen3-TTS-12Hz-1.7B-CustomVoice",
    device_map="cuda:0",
    dtype=torch.bfloat16,
    attn_implementation="sdpa",
)

print("Supported speakers:", model.get_supported_speakers())
```

说明：首次加载会从 Hugging Face 下载数 GB 的模型权重，需要等待数分钟。

---

## 生成 Ryan 英文语音

```python
import soundfile as sf
from IPython.display import Audio, display

text = "Hello! This is Ryan speaking through Qwen three TTS from Google Colab."

wavs, sample_rate = model.generate_custom_voice(
    text=text,
    language="English",
    speaker="Ryan",
    instruct="Speak clearly, warmly, and with an upbeat technical-demo tone.",
)

output_path = "qwen3_tts_ryan.wav"
sf.write(output_path, wavs[0], sample_rate)
print(f"Wrote {output_path} at {sample_rate} Hz")
display(Audio(wavs[0], rate=sample_rate))
```

成功标志：单元格中出现音频播放器，左侧 **Files** 面板中出现 `qwen3_tts_ryan.wav`。

---

## 从 Colab 取回音频：优先存入 Google Drive

### 为什么不用浏览器 Download

Colab 左侧 **Files → 文件右侧 ⋮ → Download** 是正常的人工下载路径。

但在自动化环境中，浏览器原生下载可能无法被工具可靠地接收/保存到沙盒；`google.colab.files.download()` 同样会触发浏览器端下载。因此，更可靠的中转方式是挂载 Google Drive，然后复制文件到 My Drive。

### 1. 挂载 Google Drive

```python
from google.colab import drive
drive.mount("/content/drive")
```

运行后按页面提示完成 Google 授权。成功输出必须是：

```text
Mounted at /content/drive
```

注意：左侧 Files 面板显示 **Unmount Drive** 只表示界面连接状态；务必以 runtime 的 `Mounted at /content/drive` 输出为准。

### 2. 复制音频到 My Drive

```python
import shutil, os

src = "/content/qwen3_tts_ryan.wav"
dst = "/content/drive/MyDrive/qwen3_tts_ryan.wav"

shutil.copy2(src, dst)
print("COPIED", dst, os.path.getsize(dst))
```

本次成功输出：

```text
COPIED /content/drive/MyDrive/qwen3_tts_ryan.wav 234284
```

文件随后位于 Google Drive 的 **My Drive** 根目录，可从 Drive 下载、分享或供其他工具读取。

---

## 常见问题与检查

### `No GPU detected`

回到 **Runtime → Change runtime type**，选择 GPU，并重新连接。

### 模型加载慢或内存吃紧

- 首次权重下载会较慢。
- T4 有 16 GB VRAM；使用 `torch.bfloat16` 与 `attn_implementation="sdpa"`。
- 如果 1.7B 模型发生显存不足，先用 `Qwen/Qwen3-TTS-12Hz-0.6B-CustomVoice` 验证流程。

### `find: '/content/drive': No such file or directory`

说明 runtime 还没有真正挂载 Drive。重新运行 `drive.mount('/content/drive')`，完成授权，并确认显示 `Mounted at /content/drive`。

### 复制路径不存在

先确认源文件：

```python
!ls -lh /content/qwen3_tts_ryan.wav
```

确认 Drive 挂载后再复制。

---

## 最小可复现顺序

1. 上传本地 `.ipynb`。
2. 选择 T4 GPU。
3. 安装 `qwen-tts soundfile`。
4. 加载 1.7B CustomVoice。
5. 用 Ryan 生成 WAV。
6. 播放器确认音频可用。
7. `drive.mount('/content/drive')` 并授权。
8. `shutil.copy2` 到 `/content/drive/MyDrive/qwen3_tts_ryan.wav`。
9. 用文件大小输出确认复制成功。
