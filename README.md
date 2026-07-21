# 🏎️ F1 Distributed Processing

A real-time, distributed streaming pipeline that ingests subsecond Formula 1 telemetry data, performs anomaly detection to flag off-track events using geospatial indexing, and streams live alerts to a modern frontend.

---

## 🎯 Project Overview

Modern Formula 1 relies heavily on high-frequency telemetry. This project demonstrates a scalable backend architecture designed to handle that firehose of data in real-time. It ingests live F1 car coordinates, processes them through a distributed message broker, and mathematically determines if a car has exceeded track limits (an "off-track anomaly") by comparing its position against an ideal racing line.

**Key Features:**
- **High-Throughput Ingestion**: Polls subsecond API endpoints and distributes the load via Kafka.
- **Ordered Stream Processing**: Topics are strictly partitioned by Driver ID to ensure coordinate sequences are processed in order.
- **Geospatial Anomaly Detection**: Uses Redis `GEOSEARCH` to evaluate a car's distance from the "gold standard" track boundaries.
- **Real-Time Delivery**: Pushes instantaneous anomaly alerts to connected web clients via WebSockets.

---

## 🛠️ Tech Stack

- **Python**: Core data ingestion and stream processing logic.
- **Apache Kafka**: Distributed event streaming platform for buffering and routing high-frequency telemetry.
- **Redis**: In-memory data store used for sub-millisecond geospatial (`GEOADD`, `GEOSEARCH`) track limit detection.
- **FastAPI / Node.js**: WebSocket backend for broadcasting live alerts.
- **React**: Interactive frontend dashboard visualizing live track events.

---

## 🏗️ Architecture Design

1. **Producer Engine**: A Python script continuously polls the OpenF1 `/v1/location` API, capturing coordinate deltas and publishing them to the `f1_telemetry_location` Kafka topic.
2. **Message Broker**: Kafka guarantees high availability and ordered processing by partitioning the incoming location stream by `driver_id`.
3. **Consumer & Detection**: 
   - *Initialization*: Fetches the fastest lap of the session to map the ideal racing line, scaling `X, Y` coordinates into valid longitudes/latitudes, and loading them into Redis.
   - *Processing*: Kafka consumers read the live stream, scale incoming coordinates, and execute a `GEOSEARCH` against the Redis track boundaries. If the search is empty, the car is off-track.
4. **WebSocket Broadcasting**: Off-track events are published to a secondary Kafka topic (`f1_alerts`), picked up by a WebSocket server, and broadcasted to clients.
5. **Live Dashboard**: A React frontend connects to the WebSocket stream to display anomalies in real-time.

---

## 🚀 Getting Started

*(Instructions for cloning, setting up `docker-compose`, and running the pipeline locally will be added as the infrastructure is finalized.)*