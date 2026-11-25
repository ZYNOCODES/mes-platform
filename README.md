# MES Platform: Manufacturing Execution System for TV production

A full Manufacturing Execution System that follows every television from its first component scan to the pallet handed to logistics. It covers planning, unit traceability, quality, repair, sampling, downtime, palletization, label printing and live shop-floor displays, across three production lines.

The platform is a monorepo of **nine applications**: a web stack (React, Node.js, PostgreSQL) and a set of PyQt5 workstations and Python tools that talk to scanners and label printers.

> **Structure only.** The source code is private. Source files here are intentionally empty so you can see how the project is organized; the READMEs and screenshots are complete. Setup and test commands document the private codebase.
>
> **About this repository.** This is a public portfolio copy. Company names, logos, addresses, network addresses, secrets and git history were removed, and every screenshot shows **fictional demo data** running in the real applications. The UI is in French because it was designed for French-speaking operators.

![Live TV display](docs/screenshots/web/display-tv.png)

## Contents

- [What it solves](#what-it-solves)
- [Applications](#applications)
- [Architecture](#architecture)
- [The life of a unit](#the-life-of-a-unit)
- [Screenshots](#screenshots)
- [Engineering highlights](#engineering-highlights)
- [Run it locally](#run-it-locally)
- [Tests](#tests)
- [Repository layout](#repository-layout)

## What it solves

| Shop-floor question | How the platform answers it |
| --- | --- |
| Where is this TV and what happened to it? | Each unit links its panel, motherboard and TV serials to every scan, defect, repair and pallet |
| Are we on target right now? | Plans drive live hourly cadence, target gap and progress on dashboards and wall displays |
| Which defects cost us the most? | Defects, corrective actions and sources feed quality analytics and repair timing |
| Why did the line stop? | Idle time between scans opens a downtime record; staff add the department and reason |
| Can this pallet leave? | A pallet moves one way: `open`, `pending`, `validated`, `completed` |

## Applications

| Application | Type | Role |
| --- | --- | --- |
| [web/MES_server_side](web/MES_server_side/README.md) | Node.js, Express, Sequelize | Main API: accounts, planning, stock, quality, repair, sampling, statistics, real-time events, migrations |
| [web/MES-client-side](web/MES-client-side/README.md) | React 19, Vite | Role-based web application, dedicated workspaces and TV displays |
| [web/MES-Daemon](web/MES-Daemon/README.md) | Node.js, Express, Socket.IO | Shop-floor API: scan processing, serial generation, labels, pallets, floor monitor |
| [MES-End-Assemblage](MES-End-Assemblage/README.md) | PyQt5 desktop | Station 3: end of assembly and serial label printing |
| [MES-LiberationApp](MES-LiberationApp/README.md) | PyQt5 desktop | Station 13: conform / non-conform release and ticket printing |
| [MES-Pallet-Manager](MES-Pallet-Manager/README.md) | PyQt5 desktop | Station 15: normal and quality pallets |
| [MES-Pallet-To-Logistics](MES-Pallet-To-Logistics/README.md) | PyQt5 desktop | Final hand-over of validated pallets |
| [MES-Barecode-Script-master](MES-Barecode-Script-master/README.md) | Python service | Raspberry Pi gateway from serial scanners to the daemon |
| [MES-Printers-master](MES-Printers-master/README.MD) | Python CLI | Converts PDF labels to ZPL for network printers |

## Architecture

```mermaid
flowchart LR
  subgraph Floor[Shop floor]
    SC[Barcode scanners] --> PI[Raspberry Pi gateway]
    EA[End-Assemblage]
    LB[LiberationApp]
    PM[Pallet-Manager]
    PL[Pallet-To-Logistics]
  end
  subgraph Services
    D[MES-Daemon]
    S[MES_server_side]
    DB[(PostgreSQL)]
  end
  subgraph People
    W[React web app]
    TV[TV, quality and pallet displays]
    MON[Floor monitor]
  end
  PI --> D
  EA --> D
  LB --> D
  PM --> D
  PL --> D
  D --> DB
  S --> DB
  D -- signed internal notifications --> S
  D -- PDF label --> MP[mesp CLI] --> PR[Label printers]
  S -- REST and Socket.IO --> W
  S -- Socket.IO --> TV
  D -- Socket.IO --> MON
```

The daemon and the server are separate processes on one database. Scan traffic keeps working while the web API restarts, and each service can be deployed on its own.

## The life of a unit

```mermaid
flowchart LR
  A[1 Panel scan] --> B[2 Motherboard scan] --> C[3 Assembly end and label]
  C --> D[4 Test start] --> E[5-10 Test stations] --> F{13 Liberation}
  F -- conform --> G[15 Pallet]
  F -- conform, selected --> S[Sampling test] --> QP[Quality pallet]
  F -- non conform --> R[14 Repair] --> D
  G --> H[Quality validation]
  QP --> H
  H --> I[Logistics hand-over]
```

| Station | Purpose |
| --- | --- |
| 1, 2 | Register the panel (DALL) and motherboard serials |
| 3 | Close assembly, allocate the TV serial, print its label |
| 4 | Start testing |
| 5 to 10 | Test posts, scanned only when a defect is found |
| 13 | Liberation: conform units get a ticket, others go to repair |
| 14 | Repair: defects, sources, corrective actions, waiting and repair time |
| 15 | Pallet scan |
| 16 | Panel exit scan |

TV serials follow `YYMM + model + family + lot(3) + market + sequence`. The rules are in [CODIFICATION_ARCHITECTURE.md](web/MES_server_side/docs/CODIFICATION_ARCHITECTURE.md).

## Screenshots

### Management

| Hourly cadence | Executive view |
| --- | --- |
| ![](docs/screenshots/web/dashboard-realtime.png) | ![](docs/screenshots/web/dashboard-executive.png) |
| **Quality dashboard** | **Defect analytics** |
| ![](docs/screenshots/web/dashboard-quality.png) | ![](docs/screenshots/web/quality-analytics.png) |
| **Production plans** | **Plan details** |
| ![](docs/screenshots/web/production-plans.png) | ![](docs/screenshots/web/plan-details.png) |
| **Units** | **Unit traceability** |
| ![](docs/screenshots/web/products.png) | ![](docs/screenshots/web/product-traceability.png) |

### Wall displays and floor monitor

| Pallet display | Quality display |
| --- | --- |
| ![](docs/screenshots/web/display-pallet.png) | ![](docs/screenshots/web/display-quality.png) |
| **Floor monitor** | **Line and device view** |
| ![](docs/screenshots/web/daemon-monitor.png) | ![](docs/screenshots/web/daemon-floor.png) |

### Workspaces

| Repair queue | Sampling test |
| --- | --- |
| ![](docs/screenshots/web/repair-pending.png) | ![](docs/screenshots/web/sampling-test.png) |
| **Pallet validation** | **Offline label printing** |
| ![](docs/screenshots/web/pallet-validation.png) | ![](docs/screenshots/web/offline-printing.png) |

### Desktop stations

| End of assembly | Liberation |
| --- | --- |
| ![](docs/screenshots/desktop/end-assembly-printed.png) | ![](docs/screenshots/desktop/liberation-scan.png) |
| **Pallet manager** | **Pallet to logistics** |
| ![](docs/screenshots/desktop/pallet-manager-station.png) | ![](docs/screenshots/desktop/logistics-pallet.png) |

More images are in [docs/screenshots](docs/screenshots).

## Engineering highlights

- **Concurrency.** Scans and pallet transitions run in database transactions with row locks; advisory locks protect serial allocation, and a partial unique index allows only one open pallet per line.
- **Traceability.** Scan events, plan modifications and sampling audit rows keep history. Records are soft-deleted.
- **Explicit migrations.** The API never changes the schema on startup. `db:deploy` previews migrations, checks integrity and refuses to continue on blockers.
- **Real time.** Socket.IO pushes to dashboards and displays; the daemon informs the server through authenticated internal calls.
- **Layered access.** JWT with role checks for people, per-station secrets for workstations, managed passwords for protected actions, and hashed revocable links for wall displays.
- **Hardware resilience.** Scanner reconnection, duplicate-scan protection, network and printer guards, print retry, on Windows and Ubuntu.
- **Packaging.** PyInstaller executables and Debian packages for the stations.

## Run it locally

Requirements: Node.js 22, PostgreSQL 16, Python 3.10+ for the desktop stations.

```bash
createdb mes_demo

# 1. API
cd web/MES_server_side
npm ci
cp .env.example .env            # set PORT=8000, DB_*, JWT_*, INTERNAL_SECRET
npm run db:deploy:apply         # schema, roles, lines, stations
npm run db:seed-demo            # fictional data (local databases named *demo* only)

# 2. Web application, served by the API
cd ../MES-client-side
npm ci
cp .env.example .env            # VITE_API_URL=http://localhost:8000
npm run build -- --outDir ../MES_server_side/public/dist --emptyOutDir
cd ../MES_server_side && node server.js

# 3. Shop-floor daemon (second terminal)
cd web/MES-Daemon
npm ci
cp .env.example .env            # PORT=8001, same DB and INTERNAL_SECRET
node server.js
```

Open `http://localhost:8000`. Every demo account uses the password `Demo12345`:

| User | Role |
| --- | --- |
| `demo.manager` | Production manager |
| `demo.planner` | Planning manager |
| `demo.quality` | Quality inspector |
| `demo.repair` | Repairer |
| `demo.validator` | Pallet validator |
| `demo.sampler` | Sampler |
| `demo.warehouse` | Warehouse manager |
| `demo.config` | Warehouse configurator |
| `demo.printer` | Offline label printing |

These accounts exist only in the demo database created by the seed script.

The screenshots were taken with the demo stack's clock set to mid-afternoon so that the current shift contains data. Desktop stations used a virtual serial scanner and a virtual printer.

## Tests

| Application | Command |
| --- | --- |
| Server | `cd web/MES_server_side && npm test` |
| Daemon | `cd web/MES-Daemon && npm test` |
| End-Assemblage | `cd MES-End-Assemblage && python -m unittest discover -s tests` |
| Liberation | `cd MES-LiberationApp && python -m unittest test_cross_platform test_scanner` |
| Pallet Manager | `cd MES-Pallet-Manager && python -m unittest test_cross_platform test_pallet_ticket` |
| Pallet to Logistics | `cd MES-Pallet-To-Logistics && python -m unittest discover -s tests` |

## Repository layout

```text
web/
  MES_server_side/            main API, migrations, design documents
  MES-client-side/            React application
  MES-Daemon/                 shop-floor API and floor monitor
MES-End-Assemblage/           station 3
MES-LiberationApp/            station 13
MES-Pallet-Manager/           station 15
MES-Pallet-To-Logistics/      logistics hand-over
MES-Barecode-Script-master/   Raspberry Pi scanner gateway
MES-Printers-master/          PDF to ZPL printing
docs/screenshots/             real screenshots, demo data
```

## Usage

Published for portfolio and presentation purposes. No license is granted for reuse.
