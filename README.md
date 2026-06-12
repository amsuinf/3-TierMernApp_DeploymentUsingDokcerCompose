# 3-Tier MERN Application — Containerized with Docker Compose

A full-stack MERN (MongoDB, Express, React, Node.js) application containerized from scratch and deployed as a multi-container setup using Docker Compose. Doing this project, I learned how a plain source repository — with no Docker support — can be containerized service by service, networked together, and finally orchestrated with a single `docker compose up`.

## Architecture Design

This is a classic three-tier application:

```
┌─────────────┐      ┌──────────────────┐      ┌─────────────┐
│   React     │ ───► │  Node / Express  │ ───► │   MongoDB   │
│  (Frontend) │      │    (Backend)     │      │ (Database)  │
│  Port 5173  │      │    Port 5050     │      │ Port 27017  │
└─────────────┘      └──────────────────┘      └─────────────┘
     Tier 1                 Tier 2                  Tier 3
  Presentation           Business Logic              Data
```

- **Frontend** — React UI where the user interacts with the app
- **Backend** — Express server exposing REST APIs and handling business logic
- **Database** — MongoDB storing application records, persisted via a Docker volume

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React |
| Backend | Node.js, Express |
| Database | MongoDB |
| Containerization | Docker (separate Dockerfiles per service) |
| Orchestration | Docker Compose |

## Project Structure

```
.
├── frontend/
│   ├── Dockerfile
│   └── src/
├── backend/
│   ├── Dockerfile
│   ├── server.js
│   ├── db/connection.js
│   └── routes/record.js
├── docker-compose.yaml
└── README.md
```

## What This Project Covers

- Writing **Dockerfiles** for both the React frontend and the Node backend
- Building and running each container manually first (`docker build` / `docker run`) to understand the moving parts
- Creating a **custom Docker network** so containers resolve each other by name
- Running **MongoDB as a container** with a **volume mount** so data survives container restarts
- Replacing all the manual steps with a single **`docker-compose.yaml`** — services, network, volume, environment variables, and startup order in one declarative file

## Prerequisites

- Docker Engine and Docker Compose V2 installed (`docker compose version` to verify)
- Git

## Running the Application

```bash
# 1. Clone the repository
git clone <your-repo-url>
cd <repo-folder>

# 2. Start everything
docker compose up -d

# 3. Verify all three containers are running
docker compose ps
```

Then open:

- Frontend → http://localhost:5173
- Backend API → http://localhost:5050

To stop:

```bash
docker compose down        # stop and remove containers
docker compose down -v     # also remove the MongoDB volume (wipes data)
```

## Running Without Compose (manual steps)

For reference, the same setup built by hand:

```bash
# Create a shared network
docker network create mern

# MongoDB with a volume
docker run -d --name mongodb --network mern -p 27017:27017 \
  -v mongo-data:/data/db mongo:latest

# Backend
docker build -t mern-backend ./backend
docker run -d --name backend --network mern -p 5050:5050 mern-backend

# Frontend
docker build -t mern-frontend ./frontend
docker run -d --name frontend --network mern -p 5173:5173 mern-frontend
```

Compose collapses all of the above into one file and one command — that contrast is the core lesson of this project.

## Key Learnings

- **Container networking**: containers on the same user-defined network communicate using service/container names instead of IPs (e.g., the backend connects to `mongodb://mongodb:27017`)
- **Volumes**: database data must live in a volume, not the container's writable layer, or it disappears on `docker rm`
- **Compose as documentation**: the `docker-compose.yaml` doubles as living documentation of the app's topology
- **Docker Compose V2 syntax**: `docker compose` (plugin) rather than the legacy `docker-compose` binary

## Credits

Based on the MERN containerization tutorial from Abhishek Veeramalla's *DevOps Zero to Hero* series. All implementation done hands-on.
