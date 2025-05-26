# ELS Backend System Design

## Overview

This document outlines the architecture, components, and data models used to build a USA Electronic Logging System (ELS) backend. It supports:

* Duty status tracking
* Log storage
* Real-time location tracking
* Violation detection
* Admin dashboard access
  
---

## Architecture

```
Mobile App (Bluetooth + GPS)
        |
     REST API / WebSocket
        |
+-----------------------------+
|         Backend API         |
|  - Driver/Auth Service      |
|  - Duty Log Service         |
|  - Violation Engine         |
|  - Real-time Socket Server  |
+-----------------------------+
        |
      MongoDB
        |
  Admin Dashboard (Web)
```

---

## Tech Stack

| Layer         | Technology                    |
| ------------- | ----------------------------- |
| API Framework | NestJS (Node.js)              |
| Database      | MongoDB                       |
| Realtime Comm | WebSocket (Socket.IO)         |
| Auth          | JWT + Role-based              |
| Storage       | AWS S3                        |
| Monitoring    | Grafana                       |
| Deployment    | Docker, Kubernetes            |

---

## Core Modules

### 1. Authentication & Authorization

* JWT token-based
* Role: `driver`, `admin`
* Middleware to guard routes

### 2. Driver & Vehicle Management

* Assign drivers to vehicles
* Retrieve current assignments

### 3. Duty Logs

* Capture log: status, start time, end time, location
* Log can be inserted manually (driver action) or automatically (via mobile logic)

### 4. Device Data Endpoint

* Receives data from mobile:

  * speed, odometer, engine state
  * GPS coordinates
  * Triggers log transitions (e.g., engine ON → ON\_DUTY)

### 5. Violation Engine

* Rules:

  * 11-hour driving
  * 14-hour shift
  * 30-minute break
  * 70-hour weekly limits
* Triggered at every log update

### 6. Realtime Location Tracking

* WebSocket-based
* Emits driver's current location and duty status
* Used by admin dashboard for live monitoring

### 7. Log Export

* Export duty logs to CSV
* FMCSA-style daily log formatting

---

## Database Models (Sample)

### Driver

```ts
{
  id: ObjectId,
  name: String,
  licenseNumber: String,
  hosCycle: "US_70",
  currentStatus: "OFF" | "ON" | "DRIVING" | "SLEEPER"
}
```

### Vehicle

```ts
{
  id: ObjectId,
  vin: String,
  plateNumber: String,
  assignedDriverId: ObjectId,
}
```

### DutyLog

```ts
{
  id: ObjectId,
  driverId: ObjectId,
  status: "OFF" | "ON" | "DRIVING" | "SLEEPER",
  startTime: ISODate,
  endTime: ISODate | null,
  location: { lat: Number, lng: Number },
  isViolation: Boolean
}
```

### DeviceData

```ts
{
  driverId: ObjectId,
  timestamp: ISODate,
  speed: Number,
  engineOn: Boolean,
  odometer: Number,
  location: { lat: Number, lng: Number }
}
```

### Violation

```ts
{
  driverId: ObjectId,
  type: "11_HOUR" | "14_HOUR" | "BREAK" | "CYCLE",
  occurredAt: ISODate,
  details: String
}
```

---

## API Endpoints (Examples)

### `POST /api/logs`

* Upload duty log (from mobile)

### `GET /api/logs/:driverId`

* Get logs for a driver (admin view)

### `POST /api/device-data`

* Upload vehicle/device telemetry data

### `GET /api/violations/:driverId`

* List violations

### `GET /api/export/logbook/:driverId`

* Download driver’s FMCSA logbook CSV

---

## Security

* All endpoints secured by JWT
* Rate limiting to avoid abuse
* Validation middleware for all inputs

---

## Monitoring & Logging

* REST and WebSocket events are logged
* Prometheus for system metrics
* Sentry for runtime errors

---

## Deployment Notes

* Use Docker for containerization
* Deploy to AWS/GCP/DigitalOcean
* Setup daily DB backups
---
