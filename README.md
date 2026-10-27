# Live Transcription (Windows Desktop)

[![Platform](https://img.shields.io/badge/Platform-Windows%2010%20%7C%2011%20(x64)-blue.svg)](#system-requirements)
[![Inference Engine](https://img.shields.io/badge/Engine-Faster--Whisper%20(CTranslate2)-orange.svg)](#features)
[![UI](https://img.shields.io/badge/GUI-PySide6%20(Qt)-green.svg)](#features)
[![Privacy](https://img.shields.io/badge/Privacy-100%25%20Offline%20%26%20Local-success.svg)](#privacy--security)

A high-performance, zero-cloud live speech transcription Windows desktop application powered by **Faster-Whisper (CTranslate2)** and **PySide6 (Qt)**. 

Live Transcription captures real-time system audio (WASAPI loopback from conference calls, meetings, media, or browsers) or microphone input, transcribes speech locally with near real-time latency, displays the live transcript on a modern desktop dashboard, and streams live transcript events over **USB (ADB)**, **Wi-Fi / LAN**, or **Bluetooth** to companion devices (such as Android smartphones).

---

## Architecture Overview

```text
               +--------------------------------------------------------+
               |                  Windows Host System                   |
               |                                                        |
               |   [System Audio / Speakers / WASAPI Loopback / Mic]     |
               |                           │                            |
               |                           ▼                            |
               |            [16 kHz Audio Capture Engine]               |
               |                           │                            |
               |                           ▼                            |
               |         [Faster-Whisper + Silero VAD Filter]           |
               |           (CTranslate2 CPU INT8 / CUDA FP16)           |
               |                           │                            |
               |        ┌──────────────────┴──────────────────┐         |
               |        ▼                                     ▼         |
               |  [Desktop GUI]                     [Transport Publisher]
               |  • Real-Time VU Meter              • USB Server (8765) |
               |  • In-Place Partial Text           • Wi-Fi Server (8767)|
               |  • Timestamped Segments            • Bluetooth RFCOMM  |
               +--------│─────────────────────────────────────┬─────────+
                        │                                     │
                        ▼                                     ▼
                [Desktop Display]                     [Android Companion]
```

---

## Features

- **100% Local & Zero-Cloud Processing**  
  All speech recognition is executed directly on your PC using Faster-Whisper. Audio and transcripts never leave your machine, ensuring full confidentiality for confidential meetings, technical interviews, and private calls.

- **System Audio Loopback (WASAPI)**  
  Directly captures internal system audio playing through headphones or speakers without requiring virtual audio cables, stereo mix hacks, or third-party drivers.

- **Real-Time Streaming PySide6 Interface**  
  - **In-Place Live Text**: Real-time partial transcript preview (`LIVE...`) updating continuously without duplicate line clutter.
  - **Timestamped History**: Finalized segments formatted with millisecond precision (`[HH:MM:SS.mmm]`).
  - **Live VU Meter**: Responsive audio amplitude bar reflecting incoming sound levels.
  - **Device Selector**: Quick dropdown to select or hot-swap input/output audio devices.
  - **Export & Manage**: Instant "Copy All" formatted transcript to clipboard and "Clear Display" controls.

- **Multi-Transport Streaming Server**  
  - **USB Mode (ADB)**: Ultra-low latency (<5 ms) transmission over an ADB reverse tunnel (`ws://127.0.0.1:8765`). Built-in ADB detection and one-click tunnel setup.
  - **Wi-Fi / Local Network**: High-throughput WebSocket server (`ws://0.0.0.0:8767`) with automatic UDP service discovery on port `8766` and optional pairing token authentication.
  - **Bluetooth Classic**: RFCOMM Serial Port Profile (SPP, Channel 1) for direct wireless pairing without router dependencies.

- **Standalone Portable Distribution**  
  Packaged as a precompiled Windows binary. No Python runtime, external compiler, or C++ dependencies required to run.

---

## Repository Structure

```text
LiveTranscription/
├── LiveTranscription.exe      # Main compiled Windows executable
├── _internal/                 # Bundled libraries, CTranslate2, PySide6, & dependencies
│   ├── .env.example           # Example configuration file
│   └── ...                    # C++ DLLs, ONNX runtime, and Python site-packages
└── README.md                  # Project documentation
```

---

## System Requirements

| Component | Minimum Specification | Recommended Specification |
|---|---|---|
| **Operating System** | Windows 10 (64-bit) / Windows 11 | Windows 11 (64-bit) |
| **CPU** | Intel Core i3 / AMD Ryzen 3 (4 cores) | Intel Core i5/i7 or AMD Ryzen 5/7 (6+ cores) |
| **RAM** | 4 GB | 8 GB or higher |
| **GPU (Optional)** | None (runs on CPU with INT8 quantization) | NVIDIA GPU with CUDA support for accelerated FP16 inference |
| **Audio** | Any WASAPI-compatible soundcard / output device | Standard audio interface / headset |

---

## Getting Started

### 1. Launching the Application

Simply launch `LiveTranscription.exe`:
- **Via Windows Explorer**: Double-click `LiveTranscription.exe`.
- **Via Command Prompt / PowerShell**:
  ```powershell
  .\LiveTranscription.exe
  ```

### 2. Configuration (`.env`)

To customize settings (such as Whisper model size, inference device, audio sample rates, or ports), copy `_internal\.env.example` to the root folder as `.env` and edit it:

```ini
# Server Host & USB Port
HOST=127.0.0.1
PORT=8765

# Faster-Whisper Model Settings
# Options: tiny, tiny.en, base, base.en, small, small.en, medium, medium.en, large-v3
MODEL_SIZE=tiny.en

# Device: cpu or cuda
DEVICE=cpu

# Compute type: int8 (recommended for CPU), float16 (for NVIDIA GPUs), float32
COMPUTE_TYPE=int8

# Transcription language (e.g., en, es, fr, or auto for automatic detection)
LANGUAGE=en

# Audio Capture Settings
SAMPLE_RATE=16000
CHUNK_DURATION_SEC=1.0
SILENCE_THRESHOLD=0.005
VAD_FILTER=true
MIN_SILENCE_DURATION_MS=500

# Audio Device substring match (leave empty for system default playback device)
DEVICE_NAME=
```

### 3. Command-Line Options

You can also pass arguments directly from the command line:

```powershell
.\LiveTranscription.exe [OPTIONS]
```

| Argument | Description | Default |
|---|---|---|
| `--device <name>` | Match specific audio device name/substring | Default playback |
| `--model <size>` | Faster-Whisper model (`tiny.en`, `base.en`, `small.en`, `medium.en`, `large-v3`) | `tiny.en` |
| `--language <code>` | Transcription language (`en`, `es`, `fr`, `auto`) | `en` |
| `--usb-port <port>` | USB transport WebSocket port | `8765` |
| `--wifi-port <port>` | Wi-Fi transport WebSocket port | `8767` |
| `--wifi-host <ip>` | Wi-Fi bind host IP address | `0.0.0.0` |
| `--mode <mode>` | Transport mode: `all`, `usb`, `wifi`, `bluetooth` | `all` |
| `--token <secret>` | Pairing authentication token for Wi-Fi mode | None |
| `--log-text` | Print transcript text to console in addition to UI | Disabled |

---

## Android Companion App Setup

If you are pairing this application with the **InterviewTranscriber Android Application**:

### Mode A: USB Connection (Fastest & Most Stable)
1. Connect your Android phone to the PC with a USB cable.
2. Ensure **USB Debugging** is turned on in Android Developer Options.
3. In the Windows Live Transcription GUI:
   - Go to the **USB** connection tab.
   - Click **"Establish Reverse Tunnel"** (executes `adb reverse tcp:8765 tcp:8765`).
4. On the Android phone app:
   - Select **USB** mode and tap **Connect**.

### Mode B: Wi-Fi / Local Network
1. Connect both your PC and Android phone to the same Wi-Fi network (or mobile hotspot).
2. Allow incoming traffic on TCP port `8767` and UDP port `8766` in Windows Firewall:
   ```powershell
   netsh advfirewall firewall add rule name="LiveTranscription Wi-Fi" dir=in action=allow protocol=TCP localport=8767
   netsh advfirewall firewall add rule name="LiveTranscription Discovery" dir=in action=allow protocol=UDP localport=8766
   ```
3. In the Android app:
   - Select **Wi-Fi** mode.
   - Tap **Discover Server** (or enter the PC's LAN IP address shown in the desktop UI).
   - Tap **Connect**.

### Mode C: Bluetooth
1. Pair your Android phone with Windows via standard Windows Bluetooth settings.
2. In the Android app, select **Bluetooth** mode, choose your PC, and connect over RFCOMM.

---

## Privacy & Security

- **Strictly Offline Processing**: Audio streams never leave the host machine. No audio data is uploaded to cloud APIs or remote servers.
- **No Disk Logging of Speech**: Voice data is processed transiently in memory. Transcripts are stored in volatile UI buffer memory and cleared on exit or when clicking "Clear Display".
- **Local Network Isolation**: When streaming to an external screen or phone, only finalized/partial text strings (JSON packets) are sent—raw audio waveforms are never transmitted over Wi-Fi or Bluetooth.

---

## License

This project is licensed under the [MIT License](LICENSE) (or repository owner's designated license terms).
