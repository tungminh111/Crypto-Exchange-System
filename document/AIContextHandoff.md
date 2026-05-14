# Crypto Exchange Project — Context Handoff

## Project Overview
Building a crypto exchange system. The C++ matching engine is being built separately (see Order-Matching-Engine repo). This project covers the surrounding services: User, Order, OrderBook, Notification, and the event infrastructure between them.

## My Role Preference
Act as a reviewer, not an initiator. Do NOT provide designs or implementations unless I explicitly ask. Give feedback when I share my designs.

## Tech Stack
- Backend: Go
- Frontend: React + Vite + TailwindCSS (dark trading-terminal style, multi-page)
- C++ matching engine (separate repo) — TCP input, UDP multicast L2 output, TCP matched order output, all SBE binary

## Requirements Summary
**Functional:** Login, account creation, order CRUD (LIMIT/MARKET), order history, match notifications, order book view. 200+ instruments.

**Non-functional:**
- Latency: 50ms order creation, 50ms match notification
- Scale: 10M DAU with HFT support, peak ~1M OPS (split between retail tier ~50K OPS and HFT tier ~950K OPS)
- Availability: 99.99% for services, brief failover acceptable for engine (correctness > availability)
- Consistency: Strong on balance/matching write path, eventual on display data
- Reliability: No silent order loss, idempotent notification delivery

## Architecture Decisions Made

### Services
- **UserService** — auth, balance (TigerBeetle for HFT-grade throughput), audit table (PostgreSQL), user metadata (PostgreSQL)
- **OrderService** — order CRUD, OrderDB (PostgreSQL), transactional outbox to OrderEventQueue
- **OrderEventService** — Kafka consumer/producer, bridges Kafka and C++ engine via TCP/SBE
- **OrderBookService** — UDP multicast consumer, WebSocket push to clients, recovers via EngineBackupDB snapshots
- **NotificationService** — Push notifications, UserDeviceDB (Redis)
- **OrderMatchingService** (C++) — engine, partitioned by instrument_id

### Event Infrastructure
- **OrderEventQueue** (Kafka) — partitioned by `instrument_id`, ordering required for engine ingestion
- **NotificationQueue** (Kafka) — partitioned by `user_id`, parallel delivery
- **EngineBackupDB** (MongoDB) — Kafka offset checkpoint + engine state snapshots + L2 data

### Key Patterns
- **Transactional outbox** on OrderService — order write + outbox row in single transaction, polling-based relay (not CDC, to avoid WAL bottleneck)
- **CDC from OrderDB → UserService** for audit matching (acceptable latency since not hot path)
- **Idempotency** via Kafka idempotent producer + (order_id, event_id) keys
- **Sequence IDs** on L2 UDP multicast for gap detection + snapshot recovery
- **Async checkpoint thread** in engine — takes min(latest_acked_notification, latest_stored_l2) for safe checkpoint without blocking hot loop
- **Cancel-fill race handled by state machine** — `PARTIALLY_FILLED_ON_CANCEL` is a valid terminal state, no sequence ID needed on notification path

### Balance Control Flow
- Separated balance (not engine-owned) — incompatible with instrument-partitioned engine
- TigerBeetle as durable store (not in-memory + sticky routing, for availability)
- Reservation pattern: deduct on order create → match or release on cancel
- Audit table tracks every balance mutation, matched async via OrderDB CDC

## Open Items / Next Steps
- Data model design for User & Order services (next phase)
- UI mockup created (4 pages: Login, Orders, Order Book, Notifications) — login should be separate from main nav

## Repos in Project
- Order-Matching-Engine (C++ engine, my implementation, review-only)
- Crypto-Exchange-System (Go services + frontend, we design together)
