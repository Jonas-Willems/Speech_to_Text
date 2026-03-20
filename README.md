# Flow Dictate 🎙️

A lightweight, fully offline push-to-talk dictation tool for Windows. Hold a hotkey in **any application** — Slack, VS Code, Word, your browser — release, and your speech is typed directly into it.

Powered by [OpenAI Whisper](https://github.com/openai/whisper) running locally via [faster-whisper](https://github.com/SYSTRAN/faster-whisper). Nothing ever leaves your machine.

---

## Features

- **Works in any app** — types transcribed text directly into whatever window is focused
- **Push-to-talk** — hold `F9` to record, release to transcribe
- **Fully offline** after the first model download
- **99 languages** supported with automatic detection
- **No API keys, no accounts, no cloud**

---

## Setup

### Requirements
- Windows 10/11
- Python 3.9 or newer → https://python.org/downloads

### Install dependencies

```bash
pip install faster-whisper sounddevice numpy pyautogui keyboard
```

### Run

```bash
python flow_dictate.py
```

The Whisper model downloads automatically on first run and is cached for all future runs.

---

## Usage

1. Run `flow_dictate.py` — a terminal window opens in the background
2. Click into any app (Slack, Word, browser, etc.)
3. **Hold `F9`** to record
4. **Release `F9`** — text is transcribed and typed into your app
5. `Ctrl+C` in the terminal to quit

---

## Configuration

Edit the config block at the top of `flow_dictate.py`:

| Setting | Default | Description |
|---|---|---|
| `HOTKEY` | `f9` | Key to hold for recording. E.g. `f8`, `ctrl+shift+space` |
| `MODEL` | `tiny` | Whisper model size (see below) |
| `LANGUAGE` | `None` | Auto-detect, or pin e.g. `"en"`, `"fr"`, `"de"`, `"ja"` |
| `DEVICE` | `cpu` | Use `cuda` for Nvidia GPU (much faster) |
| `TYPE_DELAY` | `0.0` | Delay between keystrokes in seconds |

### Model comparison

| Model | Transcription speed | Accuracy | Disk |
|---|---|---|---|
| `tiny` | ~1s | Good | ~150 MB |
| `base` | ~2s | Better | ~290 MB |
| `small` | ~4s | Great | ~580 MB |
| `medium` | ~8s | Excellent | ~1.5 GB |

Append `.en` (e.g. `tiny.en`) for a faster English-only version.

---

## Build a standalone .exe

No Python required on the target machine:

```bash
pip install pyinstaller
pyinstaller --onefile --console flow_dictate.py
```

The executable will be in the `dist/` folder.

### Auto-start with Windows

1. Press `Win+R`, type `shell:startup`, press Enter
2. Copy `flow_dictate.exe` (or a shortcut) into that folder

---

## Model cache location

Models are cached at:
```
C:\Users\<you>\.cache\huggingface\hub\
```

To move the cache to another drive, set the `HF_HOME` environment variable.

---

## License

MIT
