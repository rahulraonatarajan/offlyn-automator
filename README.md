# offlyn.ai Hybrid Orchestration README

This repository contains the code and configuration to integrate n8n-based multi-agent orchestration with offline agents (small language models) for hybrid workflows at offlyn.ai.

Project Overview

Offline-capable devices run a lightweight SLM runtime (e.g., llama.cpp or MLC) exposing a local HTTP API. n8n in the cloud orchestrates workflows that invoke local inference, escalate to cloud LLMs when needed, and synchronize events/tasks between offline and online environments.

Features

Offline Inference: Local SLM handles inference requests when connectivity is unavailable.

Cloud Escalation: Low-confidence or heavy workloads escalate to cloud-hosted LLMs.

Bidirectional Sync: Local events are queued and flushed to n8n; tasks from workflows are pulled to devices.

Extensible Nodes: Custom n8n node for invoking and managing offline agents.

Secure Communication: TLS and JWT-based authentication between devices and n8n.

Monitoring & Metrics: Device and workflow metrics collection for observability.

Architecture

┌───────────────┐    Internet    ┌───────────────┐
│  Devices      │◀────────────▶ │    n8n Cloud   │
│ (iOS/Android) │               │  (Orchestrator)│
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

Prerequisites

n8n (v0.XXX) deployed (Docker/Kubernetes)

Node.js and npm for custom node development

Docker for offline agent container (optional)

iOS/Android SDK for embedding offline agent

SQLite (or equivalent) for local event/task storage

Getting Started

Clone the repo:

git clone https://github.com/offlyn-ai/hybrid-orchestration.git
cd hybrid-orchestration

Install dependencies:

cd n8n-custom-node
npm install

Build the custom n8n node:

npm run build

Deploy to n8n:

Copy dist folder into your n8n nodes directory.

Restart n8n service.

Configuration

.env in n8n-custom-node:

DEVICE_API_KEY=<your_jwt_secret>

Device config:

config.json:

{
  "deviceId": "device-001",
  "n8nWebhookUrl": "https://orchestrator.offlyn.ai/webhook/offline-event",
  "authToken": "<JWT_TOKEN>"
}

Offline Agent Service

Build Docker image:

docker build -t offlyn-offline-agent ./offline-agent-service

Run locally:

docker run -d -p 8080:8080 \
  -e DEVICE_ID=device-001 \
  -e JWT_SECRET=<your_jwt> \
  offlyn-offline-agent

API Endpoints:

POST /infer → { "prompt": "..." }

POST /events → queued events

GET  /tasks → polling for tasks

Sample n8n Workflow

Webhook Trigger: Receive /webhook/offline-event.

Offline Agent Node: invokeLocalLLM.

IF confidence < 0.7 → call Cloud LLM node.

HTTP Request: Push follow-up tasks back to device.

Sync / Bridge Logic

Network Watcher in device app toggles between offline buffering and online flushing.

Queue: SQLite table events and tasks with unique IDs.

Monitoring

Expose Prometheus metrics at /metrics on device and n8n.

Grafana dashboards for:

Workflow durations

Local inference latency

Queue sizes

Testing

Unit Tests: npm test in n8n-custom-node.

Integration Tests: Simulated network scenarios in offline-agent-service/tests.

E2E: scripts/e2e.sh to spin up Docker Compose and run scenarios.

Contributing

Fork the repo

Create a feature branch

Submit a PR with tests and documentation updates
