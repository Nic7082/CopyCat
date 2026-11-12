# CopyCat: Real-Time Speech Recognition and Voice Replication System (CC-SRVR)

**Team ECEasy** | University of Virginia

CopyCat (CC-SRVR) is an end-to-end, low-latency audio pipeline designed to capture live spoken audio, transcribe speech to text, extract vocal timbre and prosody embeddings, and synthesize replicated speech in real time (**< 300 ms** target end-to-end latency).

---

## System Architecture

```text
[Microphone (16 kHz)]
       │
       ▼
┌──────────────────────────────┐
│  1. DSP Preprocessing        │  Bandpass Filter (80 Hz–7.5 kHz) ──► Spectral Gating ──► VAD
└──────┬───────────────────────┘
       │
       ├─────────────────────────────────────────┐
       ▼                                         ▼
┌──────────────────────────────┐  ┌──────────────────────────────┐
│  2. Speech Recognition (ASR) │  │  3. Speaker Encoder          │
│  Faster-Whisper / Wav2Vec2   │  │  ECAPA-TDNN / d-vectors      │
└──────┬───────────────────────┘  └──────┬───────────────────────┘
       │ (Transcribed Text)              │ (Speaker Embedding)
       └──────────────────┬──────────────┘
                          ▼
           ┌──────────────────────────────┐
           │  4. Voice Replication (TTS)  │  VITS / YourTTS + HiFi-GAN Vocoder
           └──────────────┬───────────────┘
                          ▼
                 [Replicated Audio Out]
