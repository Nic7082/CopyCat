# CopyCat: Real-Time Speech Recognition and Voice Replication System (CC-SRVR)

**Team ECEasy** | University of Virginia

CopyCat (CC-SRVR) is an end-to-end, low-latency audio processing pipeline designed to capture spoken audio, transcribe speech to text, extract speaker vocal timbre embeddings, and synthesize replicated speech in near real time (targeting **$< 300\,\text{ms}$** end-to-end latency).

---

## Table of Contents
- [System Architecture](#system-architecture)
- [Project Structure](#project-structure)
- [Prerequisites & Installation](#prerequisites--installation)
- [Usage](#usage)
- [Evaluation Metrics](#evaluation-metrics)
- [Project Roadmap](#project-roadmap)
- [Team & Roles](#team--roles)

---

## System Architecture

The system operates as an asynchronous Python pipeline divided into four modular stages:

1. **Audio Acquisition & DSP Preprocessing**
   - Captures $16\,\text{kHz}$ mono microphone audio streams via real-time buffers.
   - Applies a digital bandpass filter ($80\,\text{Hz} - 7.5\,\text{kHz}$) and spectral gating for background noise suppression.
   - Segments active speech using Voice Activity Detection (VAD) and extracts log-Mel spectrograms and Mel-Frequency Cepstral Coefficients (MFCCs).

2. **Automatic Speech Recognition (ASR)**
   - Transcribes segmented speech streams using a quantized Whisper (`faster-whisper`) or Wav2Vec 2.0 encoder-decoder model.

3. **Speaker Encoding & Voice Replication**
   - Extracts compact speaker identity embeddings (d-vectors / x-vectors) using a pretrained **ECAPA-TDNN** encoder.
   - Conditions a neural Text-to-Speech (TTS) synthesizer (**VITS** / **YourTTS**) paired with a **HiFi-GAN** neural vocoder to reconstruct the target voice.

4. **Backend Orchestration & Inference Optimization**
   - Connects all stages via `asyncio` queues in Python/PyTorch.
   - Uses FP16/INT8 quantization (ONNX Runtime / TensorRT / CTranslate2) for local CUDA GPU acceleration.

---

## Project Structure

```text
copycat-cc-srvr/
├── configs/                 # Audio, model, and quantization configuration files
├── src/
│   ├── dsp/                 # Audio I/O, bandpass filtering, spectral gating, VAD, MFCCs
│   ├── asr/                 # Streaming Whisper / Wav2Vec 2.0 transcription module
│   ├── synthesis/           # ECAPA-TDNN speaker encoder, TTS, and HiFi-GAN vocoder
│   └── pipeline/            # Async queue orchestration and latency benchmarking
├── scripts/                 # Model download and FP16/INT8 quantization scripts
├── tests/                   # Unit tests and acoustic benchmark suites
├── requirements.txt         # Python dependencies
└── README.md
```

---

## Prerequisites & Installation

### Hardware & OS Requirements
- **OS:** Linux (Ubuntu 22.04+ recommended) or Windows 10/11 with WSL2
- **GPU:** NVIDIA GPU with CUDA 11.8+ support (recommended for $< 300\,\text{ms}$ inference)
- **Audio:** Working microphone input and speaker/headphone output

### Setup

1. **Clone the repository:**
   ```bash
   git clone https://github.com/<your-username>/copycat-cc-srvr.git
   cd copycat-cc-srvr
   ```

2. **Create and activate a virtual environment:**
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

3. **Install dependencies:**
   ```bash
   pip install --upgrade pip
   pip install -r requirements.txt
   ```
   *(Note: Ensure you install the PyTorch and Torchaudio builds matching your local CUDA version.)*

4. **Download pretrained model weights:**
   ```bash
   python scripts/download_models.py
   ```

---

## Usage

### Run Real-Time Pipeline
Start the live microphone capture, transcription, and voice replication pipeline:
```bash
python -m src.pipeline.run_realtime --config configs/default.yaml
```

### Run Offline File Evaluation
Test the pipeline on a pre-recorded `.wav` file ($16\,\text{kHz}$ mono) and log latency metrics:
```bash
python -m src.pipeline.run_offline --input samples/test_speaker.wav --output outputs/replicated.wav
```

### Run Benchmarks & Tests
```bash
pytest tests/
python scripts/benchmark_latency.py
```

---

## Evaluation Metrics

The pipeline is benchmarked across three primary quantitative metrics:
- **Transcription Accuracy:** Word Error Rate ($\text{WER}$) and Character Error Rate ($\text{CER}$) across clean and noisy test sets.
- **Speaker Similarity:** Cosine similarity between source and synthesized ECAPA-TDNN speaker embeddings.
- **System Responsiveness:** End-to-end streaming latency ($\text{ms}$) from VAD speech endpoint detection to first synthesized audio chunk playback.

---

## Project Roadmap

- [x] **Phase 1 (Weeks 1–2):** Audio I/O setup, bandpass filtering, spectral gating, VAD, and log-Mel/MFCC extraction.
- [ ] **Phase 2 (Weeks 3–5):** ASR model integration and ECAPA-TDNN + VITS/YourTTS + HiFi-GAN synthesis pipeline.
- [ ] **Phase 3 (Weeks 6–7):** Asynchronous queue orchestration and FP16/INT8 quantization for $< 300\,\text{ms}$ latency.
- [ ] **Phase 4 (Week 8):** End-to-end WER, speaker similarity, and latency benchmarking + final demo video.

---

## Team & Roles

**Team ECEasy**
- **Software & Systems Integration Lead:** Nicholas "ECE" Jennings (`pby7ug@virginia.edu`)
- **Signal Processing (DSP) Engineer:** *Open / Recruiting*
- **Machine Learning & Speech Synthesis Engineer:** *Open / Recruiting*
