"""
Flow Dictate — Windows Push-to-Talk Whisper Dictation
======================================================
Hold F9 (or any key below) to record. Release to transcribe.
Text is typed directly into whatever app you have focused.

SETUP (run once in terminal):
    pip install faster-whisper sounddevice numpy pyautogui keyboard

RUN:
    python flow_dictate.py

CHANGE HOTKEY: edit HOTKEY below (e.g. 'f8', 'ctrl+shift+space')
CHANGE MODEL:  edit MODEL below — tiny.en is fastest, base.en is more accurate
"""

import os
os.environ["HF_HUB_DISABLE_SYMLINKS_WARNING"] = "1"

import threading
import tempfile
import time
import sys

import numpy as np
import sounddevice as sd
import keyboard
import pyautogui

from faster_whisper import WhisperModel

# ── Config ────────────────────────────────────────────────
HOTKEY        = "f9"           # Key to hold for recording
MODEL         = "tiny"         # tiny | base | small | medium  (multilingual, auto-detects language)
                               # append .en (e.g. "tiny.en") for English-only — slightly faster
SAMPLE_RATE   = 16000          # Required by Whisper
LANGUAGE      = None           # None = auto-detect | pin a language: "fr","es","de","ja","zh","ar","pt"...
DEVICE        = "cpu"          # "cpu" or "cuda" if you have an Nvidia GPU
COMPUTE_TYPE  = "int8"         # int8 = fast/small RAM; float16 needs GPU
TYPE_DELAY    = 0.0            # Seconds between keystrokes (0 = instant)
# ─────────────────────────────────────────────────────────

# State
recording     = False
audio_frames  = []
model         = None
lock          = threading.Lock()


def load_model():
    global model
    print(f"  Loading Whisper model '{MODEL}'... ", end="", flush=True)
    model = WhisperModel(MODEL, device=DEVICE, compute_type=COMPUTE_TYPE)
    print("ready.")


def on_press(e):
    global recording, audio_frames
    with lock:
        if not recording:
            recording = True
            audio_frames = []
            print("  ● Recording...", end="\r", flush=True)


def on_release(e):
    global recording
    with lock:
        if recording:
            recording = False
            # Copy frames so the callback thread can clear
            frames = list(audio_frames)
    if frames:
        threading.Thread(target=transcribe_and_type, args=(frames,), daemon=True).start()


def audio_callback(indata, frames, time_info, status):
    if recording:
        audio_frames.append(indata.copy())


def transcribe_and_type(frames):
    print("  ◌ Transcribing...   ", end="\r", flush=True)

    # Flatten frames to 1D float32 array
    audio = np.concatenate(frames, axis=0).flatten().astype(np.float32)

    # Whisper needs at least ~0.5s of audio
    if len(audio) < SAMPLE_RATE * 0.3:
        print("  (too short, ignored)  ")
        return

    segments, info = model.transcribe(
        audio,
        beam_size=1,
        language=LANGUAGE,         # None = auto-detect, or e.g. "fr", "es", "de"
        vad_filter=True,           # Ignore silence automatically
        vad_parameters={"min_silence_duration_ms": 300},
    )

    print(f"  (detected: {info.language}, {int(info.language_probability*100)}%)  ", end="\r", flush=True)

    text = " ".join(seg.text.strip() for seg in segments).strip()

    if not text:
        print("  (no speech detected)  ")
        return

    print(f"  ✓ {text[:80]}{'...' if len(text) > 80 else ''}  ")

    # Small pause to make sure the target window is still focused
    time.sleep(0.05)
    pyautogui.typewrite(text + " ", interval=TYPE_DELAY)


def main():
    print()
    print("  ╔══════════════════════════════════════╗")
    print("  ║        Flow Dictate  (Windows)       ║")
    print("  ╚══════════════════════════════════════╝")
    print()

    load_model()

    print()
    print(f"  Hold  [{HOTKEY.upper()}]  to record — release to transcribe.")
    print(f"  Text types into whatever app you have open.")
    print(f"  Press  Ctrl+C  to quit.")
    print()

    # Open mic stream (always open, only records when hotkey held)
    with sd.InputStream(
        samplerate=SAMPLE_RATE,
        channels=1,
        dtype="float32",
        callback=audio_callback,
        blocksize=1024,
    ):
        keyboard.on_press_key(HOTKEY, on_press)
        keyboard.on_release_key(HOTKEY, on_release)

        try:
            keyboard.wait()          # Block until Ctrl+C
        except KeyboardInterrupt:
            print("\n  Bye.")
            sys.exit(0)


if __name__ == "__main__":
    # Windows: pyautogui needs this for special chars
    pyautogui.FAILSAFE = False
    main()
