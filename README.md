# SafeCross — AI-Based Smart Crosswalk Safety System

**Full-Stack Development · Computer Vision · Real-Time Monitoring · Road Safety**

SafeCross is an AI-powered smart crosswalk monitoring platform developed as a team project at **HIT — Holon Institute of Technology**. It combines computer vision, behavioral risk classification, a real-time backend, and role-based web dashboards to help operators identify and respond to potentially dangerous situations near pedestrian crossings.

Rather than treating every detected movement as an emergency, SafeCross tracks road users across video frames and evaluates their behavior against **17 predefined risk scenarios**. Relevant alerts are delivered to a control center, where dispatchers can review and handle incidents.

## Key Features

- **AI-assisted hazard detection:** Python, OpenCV, and YOLOv8 detect and track road users in video footage.
- **Behavioral risk assessment:** 17 prioritized rules classify events as Low, Medium, or High risk using movement, proximity to the curb, and contextual signals.
- **Real-time alerts:** Socket.io broadcasts new and updated incidents to connected dashboards without requiring a page refresh.
- **Role-based access:** JWT authentication and dedicated interfaces for administrators, dispatchers, managers, and technicians.
- **Infrastructure management:** Manage monitored crossings, cameras, and LED equipment records, including locations and operational status.
- **Incident management:** Review alerts on an interactive map, handle or reopen incidents, and retain incident snapshots.
- **Safety analytics:** Server-side statistics, dangerous-crossing rankings, school-zone insights, weekly trends, and PDF reporting.
- **Maintenance workflow:** Track equipment marked as faulty and update repair status.

## System Architecture

```text
                     +-------------------------------+
                     |  React + Vite Web Application |
                     |  Role-Based Dashboards        |
                     +---------------+---------------+
                                     |
                             REST API / Socket.io
                                     |
                     +---------------v---------------+
                     |  Node.js + Express Backend    |
                     |  JWT / RBAC / Business Logic  |
                     +----+---------------+----------+
                          |               |
                 +--------v------+  +-----v----------+
                 | MongoDB Atlas |  | Cloudinary     |
                 | App Data      |  | Alert Images   |
                 +---------------+  +----------------+
                          ^
                          | Authenticated alert requests
                 +--------+--------------------+
                 | Python AI Service           |
                 | YOLOv8 + Tracking + Rules   |
                 +-----------------------------+
```

The frontend also integrates **OpenStreetMap / Leaflet** for maps and **Nominatim** for address geocoding. MongoDB change streams support live event propagation through Socket.io.

### Hazard Detection Workflow

1. The AI service processes video frames, detects objects with YOLOv8, and tracks pedestrians across frames.
2. The risk engine evaluates movement and proximity to the curb against 17 prioritized scenarios.
3. A qualifying event is submitted to the backend using a protected sensor API endpoint.
4. The backend stores the alert and its associated snapshot; connected dispatchers receive a real-time update.
5. A dispatcher reviews and resolves the incident.
6. Resolved incidents contribute to the management analytics dashboard.

Risk rules use body-height-normalized distances and speeds to reduce dependence on camera scale. **Low-risk events are logged; Medium- and High-risk events are flagged for LED warning activation.** Physical LED controller integration is a future enhancement, not a completed hardware deployment.

## User Roles

| Role | Responsibilities |
| --- | --- |
| **Admin** | Manage users, crossings, cameras, LED equipment, and infrastructure status. |
| **Dispatcher** | Monitor live alerts and the map; resolve or reopen incidents. |
| **Manager** | Review safety metrics, trends, high-risk locations, and PDF reports. |
| **Technician** | View faulty equipment and update repair status. |

## Technology Stack

| Layer | Technologies |
| --- | --- |
| **Frontend** | React, Vite, Tailwind CSS, React Router, Axios, Socket.io Client, Recharts, React-Leaflet |
| **Backend** | Node.js, Express, Socket.io, Mongoose, JWT, bcrypt |
| **Database** | MongoDB Atlas |
| **AI Service** | Python, YOLOv8 (Ultralytics), OpenCV, NumPy, FastAPI, Uvicorn |
| **Media Storage** | Cloudinary |
| **Maps & Geocoding** | OpenStreetMap, Leaflet, Nominatim |
| **Testing** | Node.js Test Runner, mongodb-memory-server, Python unittest |

## Repository Structure

```text
smart-crosswalk-safety-system/
├── README.md
├── docs/
│   ├── Project_Presentation.pdf
│   ├── Project_Summary.pdf
│   ├── Risk_Rules.pdf
│   ├── Risk_Rules_HE.pdf
│   ├── Technical_Documentation.pdf
│   └── Technical_Documentation_HE.pdf
└── src/
    ├── backend/
    │   ├── ai-service/
    │   │   ├── app.py
    │   │   ├── main.py
    │   │   ├── risk_rules.py
    │   │   ├── requirements.txt
    │   │   └── tests/
    │   ├── routes/
    │   ├── services/
    │   ├── models/
    │   ├── tests/
    │   └── package.json
    └── frontend/
        ├── src/
        └── package.json
```

The AI service is included in `src/backend/ai-service/`. The repository structure above shows the principal source directories; generated and local-only files are omitted.

## Documentation

- [`Project Summary`](docs/Project_Summary.pdf) — concise overview of the problem, solution, architecture, and capabilities.
- [`Technical Documentation`](docs/Technical_Documentation.pdf) — system requirements, architecture, APIs, security, data modeling, AI pipeline, and testing (English).
- [`Technical Documentation (Hebrew)`](docs/Technical_Documentation_HE.pdf) — Hebrew version of the technical documentation.
- [`Risk Rules`](docs/Risk_Rules.pdf) — the 17 behavioral risk scenarios and classification criteria (English).
- [`Risk Rules (Hebrew)`](docs/Risk_Rules_HE.pdf) — Hebrew version of the risk rules.
- [`Project Presentation`](docs/Project_Presentation.pdf) — project presentation and architectural overview.
- [`Project Demo Video`](https://www.youtube.com/watch?v=LR4Ud5pgH0E&t=30s) — video demonstration of the system's functionality and key features.

## Running the Project Locally

The application has a React frontend, a Node.js backend, and a separate Python AI service. The backend uses MongoDB Atlas and Cloudinary; local operation requires your own service accounts and credentials.

**Prerequisites:** Node.js and npm, Python 3.10+, a MongoDB connection, and Cloudinary credentials.

### 1. Backend

```bash
cd src/backend
npm install
```

Copy `.env.example` to `.env` in `src/backend/` and configure `PORT`, `FRONTEND_URL`, `AI_SERVICE_URL`, `MONGO_URI`, `CLOUDINARY_CLOUD_NAME`, `CLOUDINARY_API_KEY`, `CLOUDINARY_API_SECRET`, `JWT_SECRET`, and `SENSOR_API_KEY` as appropriate.

```bash
npm start
```

The default backend port in `.env.example` is `3000`.

### 2. Frontend

Open a second terminal:

```bash
cd src/frontend
npm install
npm run dev
```

Vite typically serves the frontend at `http://localhost:5173`. Check the frontend configuration for its expected backend URL.

### 3. Python AI service

Open another terminal:

```bash
cd src/backend/ai-service
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
python verify_setup.py --no-window
```

To start the FastAPI image-detection service:

```bash
uvicorn app:app --host 127.0.0.1 --port 8000
```

To analyze a local video without submitting alerts to the backend:

```bash
python main.py --source path/to/video.mp4 --no-api
```

To send alerts to the Node.js backend, omit `--no-api` after configuring the required API credentials and crosswalk/camera settings. The AI video-analysis CLI and FastAPI image endpoint are distinct entry points.

## Testing & Engineering Highlights

The repository contains backend end-to-end tests using `node:test` and `mongodb-memory-server`, as well as Python unit tests for tracking, video processing, and risk classification. Run them from their respective directories:

```bash
cd src/backend
npm run test:e2e
```

```bash
cd src/backend/ai-service
python -m unittest discover -s tests -t . -v
```

## Project Scope & Future Work

SafeCross is an academic full-stack project and prototype, **not a certified or deployed road-safety system**. Planned extensions include physical LED warning-sign integration, direct live-camera/RTSP processing, operator-feedback-driven model improvements, and municipal smart-city integration.

## Project Team

Developed as a team project at **HIT — Holon Institute of Technology** by:

- Yosef Yosef
- Netanel Kadoshi
- Or Smolarchik
- Rachel Brodsky
- Liel Babayn

**Project mentor:** Netanel Ben Hamo

---

*SafeCross explores how computer vision, real-time systems, and data-driven operations can support safer pedestrian crossings.*
