# VSign: Real-Time Multilingual Vietnamese Sign Language Recognition & Translation System

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue.svg)](https://www.python.org/)
[![Framework](https://img.shields.io/badge/PyTorch-2.0%2B-ee4c2c.svg)](https://pytorch.org/)
[![Backend](https://img.shields.io/badge/NestJS%2FFastAPI-Supported-red.svg)](https://nestjs.com/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

**VSign** is an end-to-end software and AI system designed to bridge the communication gap between Vietnamese Sign Language (VSL) users and non-signers. The system captures VSL gestures via a real-time web camera, extracts spatio-temporal features, predicts gloss sequences using **Transfer Learning**, converts glosses into natural Vietnamese sentences, translates them into multiple target languages, and synthesizes speech output via Text-to-Speech (TTS).

---

## Key Features

- **Real-Time Gesture Capture:** Streams video frames and extracts 2D/3D hand and pose landmarks via MediaPipe.
- **VSL Recognition Engine:** 
  - Dual architecture: Light Baseline (Landmarks + BiLSTM/GRU) and Transfer Learning Main (Pretrained Video/Pose Backbone).
  - Robust temporal segmentation and confidence gating to handle silence and frame noise.
- **Natural Language Post-Processing:** Translates raw VSL gloss sequences into coherent, natural Vietnamese sentences without semantic hallucination.
- **Multilingual Pivot Architecture:** Uses Vietnamese as a central pivot language to support translation to English, Japanese, Korean, Chinese, and more without retraining the core VSL recognizer.
- **Text-to-Speech (TTS) Integration:** Provides instantaneous voice responses in the selected target language.
- **Signer-Independent Evaluation:** Built-in data split and evaluation pipeline strictly designed to test generalization on unseen signers.

---

## System Architecture

VSign is designed with a modular, decoupled architecture separating UI/streaming, application business logic, AI inference, and language translation layers.

```text
┌─────────────────┐      WebSocket / HTTP      ┌──────────────────────────┐
│ Client (Web/UI) │ ─────────────────────────> │   Application Backend    │
│ Camera Capture  │ <───────────────────────── │ (Session, Auth, Logging) │
└─────────────────┘                            └────────────┬─────────────┘
                                                            │ Internal API
                                                            ▼
                                               ┌──────────────────────────┐
                                               │   AI Inference Service   │
                                               │ (Landmarks + VSL Model)  │
                                               └────────────┬─────────────┘
                                                            │ Gloss Sequence
                                                            ▼
                                               ┌──────────────────────────┐
                                               │ Language & Translation   │
                                               │ (Gloss → VN → Target/TTS)│
                                               └──────────────────────────┘
```

---

## Tech Stack

- **AI & Deep Learning:** PyTorch, OpenCV, MediaPipe, NumPy, ONNX Runtime.
- **AI Inference Service:** Python, FastAPI, WebSockets.
- **Application Backend:** NestJS / Node.js (or FastAPI), TypeScript.
- **Frontend / Client:** Next.js / React, Web Camera API, TailwindCSS.
- **Database & Storage:** PostgreSQL (Metadata/Logs), MinIO / Amazon S3 (Video Dataset & Artifacts), Redis (Cache/Queue).
- **DevOps & Infrastructure:** Docker, Docker Compose, GitHub Actions CI/CD.

---

## Project Structure

```text
VSign/
├── apps/
│   ├── client/                  # Frontend Web Application (Next.js/React)
│   ├── backend/                 # Application Backend Service (NestJS/FastAPI)
│   └── ai-service/              # Python Inference Service & Model Runners
├── packages/
│   ├── data-pipeline/           # Data collection, labeling, landmark extraction
│   └── models/                  # PyTorch model definitions (Baseline & Pretrained TL)
├── docs/                        # Project research documentations & specs
├── docker-compose.yml           # Local multi-service environment setup
├── requirements.txt             # Python dependencies
├── package.json                 # Node.js workspace management
└── README.md
```

---

## Getting Started

### Prerequisites

- **Node.js:** `v18+`
- **Python:** `v3.10+`
- **Docker & Docker Compose:** Installed and running
- **CUDA Toolkit** (Optional, for GPU-accelerated training/inference)

### 1. Clone the Repository

```bash
git clone https://github.com/your-org/VSign.index.git
cd VSign
```

### 2. Environment Configuration

Create a `.env` file in the root directory based on `.env.example`:

```bash
cp .env.example .env
```

Configure database credentials, JWT secrets, API keys for Translation/TTS services, and model paths.

### 3. Launching with Docker Compose (Recommended)

Start all system services (Database, Backend, AI Inference Service, Client) with a single command:

```bash
docker-compose up --build -d
```

- **Web Client UI:** `http://localhost:3000`
- **Backend API:** `http://localhost:4000/api`
- **AI Inference API:** `http://localhost:8000/docs`

### 4. Running AI Service Locally (Development Mode)

If you are developing or training models separately:

```bash
cd apps/ai-service
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt

# Run inference service locally
uvicorn app.main:app --reload --port 8000
```

---

## Experiments & Evaluation

VSign provides standardized evaluation scripts ensuring no data leakage across signers (`signer-independent split`).

### Run Evaluation Script

```bash
python packages/models/evaluate.py \
  --model-path artifacts/checkpoints/best_tl_model.pth \
  --test-split data/processed/test_signer_independent.json \
  --metrics accuracy macro_f1 wer latency
```

### Supported Metrics

- **Recognition Accuracy & Macro-F1:** Evaluates class balance across vocabulary items.
- **Word Error Rate (WER) / Gloss Error Rate:** Measures temporal decoding efficiency.
- **Latency (p50 / p95):** Real-time responsiveness benchmarking (FPS and end-to-end response time).

---

## API & Streaming Protocols

### Real-Time WebSocket Streaming

- **Endpoint:** `ws://localhost:4000/ws/recognition/{sessionId}`
- **Payload Input (Client → Server):** JSON object containing frame timestamps and keypoint landmarks.
- **Response Output (Server → Client):**

```json
{
  "sessionId": "sess_123456",
  "predictedGloss": "TÔI_MUỐN_UỐNG_NƯỚC",
  "confidence": 0.92,
  "vietnameseSentence": "Tôi muốn uống nước.",
  "targetTranslation": "I want to drink water.",
  "latencyMs": 145,
  "status": "STABLE"
}
```

---

## Data Privacy & Governance

VSign handles video recordings and landmark trajectories with strict adherence to data privacy standards:
- Raw video frames are processed **in-memory** and are not retained on disk by default during real-time inference.
- Dataset collection adheres to participant consent forms and anonymized `signer_id` metadata tags.

---

## Roadmap

- [x] **Phase 1:** Data Pipeline setup, MediaPipe feature extraction & Baseline Landmark + BiLSTM model.
- [x] **Phase 2:** Transfer Learning integration, Signer-Independent split benchmark.
- [ ] **Phase 3:** Continuous Sign Language Recognition (CSLR) using CTC temporal decoding.
- [ ] **Phase 4:** On-device / Edge runtime optimization via ONNX Quantization & Web Assembly (WASM).

---

## Contributing

Contributions are welcome! Please read our [CONTRIBUTING.md](CONTRIBUTING.md) guide before submitting Pull Requests.

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'feat: Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

---
