# Distributed Notification System

An event-driven notification system that dispatches notifications across multiple channels (Email, SMS, Push, In-App), built with NestJS, Kafka, PostgreSQL, and Redis.

## Features
- **Event-Driven Dispatch** — Kafka decouples notification triggering from delivery, processing Email, SMS, and Push notifications asynchronously through separate channel-specific services
- **Deduplication & Rate Limiting** — Redis prevents duplicate notification dispatch and rate-limits sends to control delivery volume and third-party API costs
- **Webhook Ingestion** — A lightweight Express.js receiver ingests delivery-status webhooks from external providers (e.g., Twilio, SendGrid) and republishes them onto Kafka for downstream processing
- **Real-Time In-App Notifications** — Delivered via WebSockets, backed by a PostgreSQL schema for templates, user preferences, and read/unread state

## Tech Stack
- **Framework:** NestJS (TypeScript) + Express.js (webhook receiver)
- **Message Broker:** Kafka
- **Database:** PostgreSQL
- **Caching / Dedup / Rate Limiting:** Redis
- **Real-time:** WebSockets
- **Auth:** JWT

## Architecture
\`\`\`
src/
├── auth/              # JWT authentication
├── notification-core/  # Event ingestion, template resolution
├── channels/
│   ├── email/           # Email delivery service
│   ├── sms/              # SMS delivery service
│   └── push/             # Push delivery service
├── webhook-receiver/    # Express.js delivery-receipt ingestion
├── in-app/               # WebSocket gateway for in-app notifications
└── main.ts
\`\`\`
*(placeholder structure — verify folder names against actual project before publishing)*

## API Endpoints
*(placeholder route names — verify against actual controllers and correct before publishing)*

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/notifications` | Trigger a new notification event (publishes to Kafka) |
| GET | `/notifications` | List a user's notifications (in-app) |
| PATCH | `/notifications/:id/read` | Mark a notification as read |
| POST | `/webhooks/:provider` | Receive delivery-status callbacks from a provider |

## Core Logic: Event Flow
A notification event is published to Kafka on trigger. Each channel (Email/SMS/Push) has its own consumer that processes events relevant to it, checks Redis for a deduplication key before dispatch, and calls the relevant third-party provider. Delivery receipts come back through the webhook receiver and are republished to Kafka for status tracking.

> **Note:** channel services currently run within the same NestJS application as separate modules rather than as independently deployed services — full deployment-level separation (each channel as its own process/container) is a planned next step, not yet implemented.