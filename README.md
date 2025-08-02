# offlyn.ai Hybrid Orchestration README

This repository contains the code and configuration to integrate **n8n-based multi-agent orchestration** with **offline agents** (small language models) for hybrid workflows at offlyn.ai.

---

## 📖 Project Overview

Offline-capable devices run a lightweight SLM runtime (e.g., `llama.cpp` or MLC) exposing a local HTTP API. n8n in the cloud orchestrates workflows that:

1. Invoke local on-device inference.
2. Escalate to cloud-hosted LLMs when needed.
3. Synchronize events/tasks between offline and online environments.

---

## 🚀 Features

* **Offline Inference**
  Local SLM handles inference requests when connectivity is unavailable.
* **Cloud Escalation**
  Low-confidence or resource-intensive requests escalate to cloud LLMs.
* **Bidirectional Sync**
  Local events queue & flush to n8n; workflows push tasks back to devices.
* **Extensible Nodes**
  Custom n8n node for invoking and managing offline agents.
* **Secure Communication**
  TLS & JWT-based authentication between devices and n8n.
* **Monitoring & Metrics**
  Device and workflow metrics collection for observability.

---

## 🏗️ Architecture

```text
┌───────────────┐    Internet    ┌───────────────┐
│   Devices     │◀────────────▶ │   n8n Cloud   │
│ (iOS/Android) │               │ (Orchestrator)│
└────┬──────────┘               └───▲───┬─────▲──┘
     │ Local HTTP API                  │     │
     ▼                                 │     │
┌───────────────┐       │              │     │
│ Offline Agent │◀──────┘              │     │
│   Service     │                      │     │
└───────────────┘                      │     │
                                        │     │
                        ┌───────────────▼─────▼───┐
                        │ External APIs & Cloud LLMs │
                        └────────────────────────────┘
```

---

## 🛠️ Prerequisites

* **n8n** (v0.XXX) deployed (Docker/Kubernetes)
* **Node.js** & **npm** for custom node development
* **Docker** (optional) for offline agent container
* **iOS/Android SDK** for embedding agent
* **SQLite** (or equivalent) for local event/task storage

---

## 🔧 Getting Started

1. **Clone the repo**

   ```bash
   git clone https://github.com/offlyn-ai/hybrid-orchestration.git
   cd hybrid-orchestration
   ```
2. **Install dependencies**

   ```bash
   cd n8n-custom-node
   npm install
   ```
3. **Build the custom n8n node**

   ```bash
   npm run build
   ```
4. **Deploy to n8n**

   * Copy `dist/` into your n8n `nodes` directory
   * Restart n8n service

---

## ⚙️ Configuration

* **.env** in `n8n-custom-node`:

  ```ini
  DEVICE_API_KEY=<your_jwt_secret>
  ```
* **Device `config.json`**:

  ```json
  {
    "deviceId": "device-001",
    "n8nWebhookUrl": "https://orchestrator.offlyn.ai/webhook/offline-event",
    "authToken": "<JWT_TOKEN>"
  }
  ```

---

## 📡 Offline Agent Service

1. **Build Docker image**

   ```bash
   docker build -t offlyn-offline-agent ./offline-agent-service
   ```
2. **Run locally**

   ```bash
   docker run -d -p 8080:8080 \
     -e DEVICE_ID=device-001 \
     -e JWT_SECRET=<your_jwt> \
     offlyn-offline-agent
   ```
3. **API Endpoints**

   * `POST /infer` → `{ "prompt": "..." }`
   * `POST /events` → queued events
   * `GET  /tasks` → poll for tasks

---

## 🔄 Sample n8n Workflow

1. **Webhook Trigger**: `/webhook/offline-event`
2. **Offline Agent Node**: `invokeLocalLLM`
3. **IF** confidence < 0.7 → Cloud LLM node
4. **HTTP Request**: Push tasks back to device

---

## 🏗️ Sync / Bridge Logic

* **Network Watcher** toggles between offline buffering & online flushing
* **Queue**: SQLite tables `events` & `tasks` with unique IDs

---

## 📊 Monitoring

* Prometheus metrics endpoint at `/metrics` (device & n8n)
* Grafana dashboards: workflow durations, inference latency, queue sizes

---

## 🧪 Testing

* **Unit Tests**: `npm test` (n8n-custom-node)
* **Integration Tests**: simulated network flaps (`offline-agent-service/tests`)
* **E2E**: `scripts/e2e.sh` (Docker Compose scenarios)

---

## 🤝 Contributing

1. Fork the repo
2. Create a feature branch
3. Submit a PR with tests & docs updates

---

## 📝 License

MIT © offlyn.ai
