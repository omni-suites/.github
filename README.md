<div align="center">

# 🌐 Omni-Suites Ecosystem
### An Enterprise Quality Engineering & Test Automation Workflow for Agile-Driven Teams

[![TypeScript](https://img.shields.io/badge/TypeScript-5.0+-3178C6?style=for-the-badge&logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![NestJS](https://img.shields.io/badge/NestJS-11.0+-E0234E?style=for-the-badge&logo=nestjs&logoColor=white)](https://nestjs.com/)
[![React](https://img.shields.io/badge/React-18.0+-61DAFB?style=for-the-badge&logo=react&logoColor=black)](https://react.dev/)
[![Playwright](https://img.shields.io/badge/Playwright-1.50+-2EAD33?style=for-the-badge&logo=playwright&logoColor=white)](https://playwright.dev/)
[![k6](https://img.shields.io/badge/Grafana_k6-0.54+-7D64FF?style=for-the-badge&logo=k6&logoColor=white)](https://k6.io/)
[![Docker Compose](https://img.shields.io/badge/Docker_Compose-v2-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://docs.docker.com/compose/)
[![Traefik](https://img.shields.io/badge/Traefik_v3-Proxy-24A1C1?style=for-the-badge&logo=traefik&logoColor=white)](https://traefik.io/)
[![ReportPortal](https://img.shields.io/badge/ReportPortal-5.15-007ACC?style=for-the-badge&logo=datadog&logoColor=white)](https://reportportal.io/)
[![Squash TM](https://img.shields.io/badge/Squash_TM-TCMS-FF6B6B?style=for-the-badge)](https://www.squashtest.com/)

<p align="center">
  A state-of-the-art cloud native platform orchestrating polyglot microservices, distributed persistence, centralized E2E and performance automation suites, automated test case synchronization with Linear and Squash TM, and Discord-based ChatOps CI/CD dispatch.
</p>

</div>

---

## 📖 Table of Contents

1. [Executive Overview & Vision](#-executive-overview--vision)
2. [Global Architecture & End-to-End Topology](#-global-architecture--end-to-end-topology)
3. [Organization Repositories Directory](#-organization-repositories-directory)
4. [The System Under Test (SUT)](#-the-system-under-test-sut)
5. [Quality Engineering & Automation Platforms](#-quality-engineering--automation-platforms)
   - [Centralized Playwright E2E & API Suites](#1-centralized-playwright-e2e--api-suites-test-suites)
   - [Centralized Grafana k6 Performance Testing](#2-centralized-grafana-k6-performance-testing-perf-suites)
   - [Test Management & Linear Webhook Sync (Squash TM)](#3-test-case-management--linear-sync-squash-tm)
   - [AI-Driven Defect Analytics & Reporting (ReportPortal)](#4-ai-driven-test-analytics--dashboards-reportportal)
6. [ChatOps & Developer Experience (Discord Bot)](#-chatops--developer-experience-discord-bot)
7. [Cloud Infrastructure, Staging VM & Ingress](#-cloud-infrastructure-staging-vm--ingress)
8. [Master Public Routing Directory](#-master-public-routing-directory)
9. [Security, Networking & Secrets Management](#-security-networking--secrets-management)
10. [Local Development & Contribution Workflow](#-local-development--contribution-workflow)

---

## 🎯 Executive Overview & Vision

Modern engineering organizations operating distributed microservices frequently suffer from **QA fragmentation**: tests are siloed inside application repositories, performance benchmarks are run inconsistently, test case management is disconnected from sprint tickets, and results are scattered across individual CI runner logs.

**Omni-Suites** solves this by establishing a centralized, enterprise-grade Quality Engineering ecosystem:
* **Decoupled Automation Platforms**: Centralized repositories for E2E/API testing ([`test-suites`](https://github.com/omni-suites/test-suites)) and load/performance testing ([`perf-suites`](https://github.com/omni-suites/perf-suites)) that run against ephemeral, staging, or production environments without modifying application source code.
* **Bi-Directional Requirement Traceability**: Issues in **Linear** labeled with `ready-for-tc` automatically synchronize with **Squash TM** via [`omni-integration`](https://github.com/omni-suites/omni-integration), auto-generating structured test cases and preserving test coverage history.
* **Unified Observability & AI Categorization**: All browser and API test executions automatically publish metrics, error logs, and failure screenshots directly to a self-hosted **ReportPortal v5.15** cluster with automated defect classification.
* **Interactive ChatOps**: Team members can execute on-demand regression suites (`/run-tests`) or stress benchmarks (`/run-perf`) directly from **Discord**, streaming real-time embeds and artifact links back into communication channels.

---

## 🏛️ The System Architecture

<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/b08c3ad4-8776-4e5c-97aa-d35c215abd27" />

## 🏛️ Global Architecture & End-to-End Topology

```mermaid
flowchart TD
    subgraph Clients ["Public Ingress & Users"]
        WebUser["End User / Browser"]
        QAUser["QA Engineer / Tester"]
        LinearWH["Linear Webhooks"]
        DiscordUser["Discord ChatOps (/run-tests, /run-perf)"]
    end

    subgraph Edge ["Central Edge Reverse Proxy (infra/traefik)"]
        TraefikProxy["Traefik v3.7 Edge Proxy<br/>Port 80 (HTTP) -> 443 (HTTPS)<br/>Auto Let's Encrypt TLS"]
    end

    WebUser -->|frontend-svc.test-suites-poc.work.gd| TraefikProxy
    QAUser -->|squash-tcms.test-suites-poc.work.gd/squash| TraefikProxy
    QAUser -->|report-portal.test-suites-poc.work.gd| TraefikProxy
    LinearWH -->|hooks.test-suites-poc.work.gd/webhooks/linear| TraefikProxy
    DiscordUser -->|hooks.test-suites-poc.work.gd/api/discord/interactions| TraefikProxy

    subgraph AppsStack ["Core Application Stack (infra/apps)"]
        subgraph FrontEnds ["UI Layer"]
            OmniClient["omni-client (:80)<br/>React 18 + Vite SPA"]
        end

        subgraph Backends ["Microservices Layer"]
            OrderSvc["order-service (:3000)<br/>NestJS + Prisma"]
            InvSvc["inventory-service (:3001)<br/>NestJS + Prisma"]
            NotifSvc["notification-service (:3002)<br/>NestJS + Prisma"]
            IntegrationSvc["omni-integration (:3003)<br/>NestJS Discord & Linear Webhook"]
        end

        subgraph DBs ["Polyglot Persistence Layer (apps-internal network)"]
            PostgresApps[("postgres-db (:5432)<br/>order_db & omni_integration_db")]
            MySQLApps[("mysql-db (:3306)<br/>inventory_db")]
            MongoApps[("mongo-db (:27017)<br/>notification_db (Replica Set rs0)")]
        end
    end

    subgraph QAStack ["Quality & Test Management Infrastructure"]
        subgraph SquashStack ["infra/squash-tm"]
            SquashApp["squash-tm (:8080/squash)<br/>Test Management System"]
            SquashDB[("squash-tm-pg (:5432)<br/>PostgreSQL 15")]
        end

        subgraph RPStack ["infra/report-portal"]
            RPGateway["report-portal-gateway (:8080)<br/>Internal Traefik Gateway"]
            RPServices["ReportPortal Microservices<br/>(API, UAT, UI, Jobs)"]
            RPDB[("postgres (:5432)<br/>PostgreSQL 18.4")]
            RPRabbitMQ[("rabbitmq (:5672)<br/>RabbitMQ 4.3")]
            RPOpenSearch[("opensearch (:9200)<br/>OpenSearch 3.8")]
            RPAnalyzer["service-auto-analyzer (:5001)<br/>AI Defect Clustering"]
        end
    end

    subgraph TestExecution ["Test Execution Engines (GitHub Actions Runners)"]
        PlaywrightEngine["Playwright Test Engine (test-suites)<br/>UI & API Tests"]
        K6Engine["Grafana k6 Engine (perf-suites)<br/>Load, Stress, Spike Tests"]
    end

    TraefikProxy --> OmniClient
    TraefikProxy --> OrderSvc
    TraefikProxy --> InvSvc
    TraefikProxy --> NotifSvc
    TraefikProxy --> IntegrationSvc
    TraefikProxy --> SquashApp
    TraefikProxy --> RPGateway

    OmniClient -.->|Browser API Requests| TraefikProxy
    OrderSvc -->|HTTP RPC :3001| InvSvc
    OrderSvc -->|HTTP RPC :3002| NotifSvc
    OrderSvc --> PostgresApps
    InvSvc --> MySQLApps
    NotifSvc --> MongoApps
    IntegrationSvc --> PostgresApps

    IntegrationSvc -->|REST API :8080/squash| SquashApp
    IntegrationSvc -->|Dispatch Workflow| PlaywrightEngine
    IntegrationSvc -->|Dispatch Workflow| K6Engine

    SquashApp --> SquashDB
    RPGateway --> RPServices
    RPServices --> RPDB
    RPServices --> RPRabbitMQ
    RPServices --> RPOpenSearch
    RPAnalyzer --> RPRabbitMQ
    RPAnalyzer --> RPOpenSearch

    PlaywrightEngine -->|Run Tests Against Services| TraefikProxy
    PlaywrightEngine -->|Report Results & Traces| RPServices
    PlaywrightEngine -->|Sync Test Runs| SquashApp
    K6Engine -->|Load & Stress Ingestion| TraefikProxy
```

---

## 🗂️ Organization Repositories Directory

| Repository | Category | Core Stack | Description |
| :--- | :--- | :--- | :--- |
| [**`omni-client`**](https://github.com/omni-suites/omni-client) | Application | React 18, Vite, TypeScript | Customer-facing e-commerce storefront SPA with real-time inventory updates and purchase notifications. |
| [**`order-service`**](https://github.com/omni-suites/order-service) | Application | NestJS, Prisma, PostgreSQL 15 | Core transactional order fulfillment microservice orchestrating inventory deduction and notifications. |
| [**`inventory-service`**](https://github.com/omni-suites/inventory-service) | Application | NestJS, Prisma, MySQL 8 | Inventory ledger and stock management service with atomic deduction routines and auto-seeding. |
| [**`notification-service`**](https://github.com/omni-suites/notification-service) | Application | NestJS, Prisma, MongoDB 6 (`rs0`) | Event notification microservice utilizing MongoDB replica set for transaction-safe alert persistence. |
| [**`omni-integration`**](https://github.com/omni-suites/omni-integration) | Integration | NestJS, Prisma, PostgreSQL 15 | Central middleware for Linear webhooks, Squash TM synchronization, and Discord slash command ChatOps. |
| [**`test-suites`**](https://github.com/omni-suites/test-suites) | Quality & QA | Playwright, TypeScript, Node.js | Centralized browser E2E and HTTP API test automation framework streaming to ReportPortal. |
| [**`perf-suites`**](https://github.com/omni-suites/perf-suites) | Quality & QA | Grafana k6, TypeScript, Webpack | Centralized load, stress, spike, and performance benchmark suites with custom reporting. |
| [**`infra`**](https://github.com/omni-suites/infra) | Platform & Ops | Docker Compose, Traefik v3 | Staging infrastructure orchestration: Traefik proxy, all business applications, Squash TM, and ReportPortal. |
| [**`.github`**](https://github.com/omni-suites/.github) | Management | GitHub Community & Profile | Central organization documentation, community health files, and platform guides. |

---

## 🛒 The System Under Test (SUT)

The business platform simulates a multi-tier e-commerce order workflow:

```text
[User clicks Buy on omni-client]
          │
          ▼
    [POST /orders] ──────────────────────────┐
          │ (HTTP RPC :3001)                 │
          ▼                                  ▼
[POST /inventory/deduct]               [POST /notifications] (HTTP RPC :3002)
          │                                  │
          ▼                                  ▼
   (Deducts stock in MySQL)          (Logs event in MongoDB replica set)
          │                                  │
          └────────────────┬─────────────────┘
                           ▼
             (Persists Order in PostgreSQL)
                           │
                           ▼
          [Order Confirmation Displayed in UI]
```

### Microservice Endpoints
* **Order Service (`:3000`)**: `POST /orders`, `GET /orders`, `GET /orders/:id`, `GET /health`
* **Inventory Service (`:3001`)**: `GET /inventory`, `POST /inventory/deduct`, `GET /health`
* **Notification Service (`:3002`)**: `GET /notifications`, `POST /notifications`, `GET /health`
* **Omni Integration (`:3003`)**: `POST /webhooks/linear`, `POST /api/discord/interactions`, `GET /health`

---

## 🧪 Quality Engineering & Automation Platforms

### 1. Centralized Playwright E2E & API Suites (`test-suites`)
* **Technology**: Playwright Test, TypeScript, Page Object Model (POM), custom domain fixtures.
* **Scope**: Multi-browser E2E testing (Chromium, Firefox, WebKit), end-to-end purchasing flows, and standalone REST API validation against microservice contracts.
* **Execution Options**: Triggered via GitHub Actions `workflow_dispatch`, scheduled nightly runs, or Discord ChatOps.
* **Direct Integration**: Native reporting into **ReportPortal** using `@reportportal/agent-js-playwright` and test status syncing into **Squash TM**.

### 2. Centralized Grafana k6 Performance Testing (`perf-suites`)
* **Technology**: Grafana k6, TypeScript, Webpack bundling, Domain-Driven Design layout (`src/core`, `src/modules`, `src/journeys`).
* **Workload Profiles**:
  * `smoke`: Quick verification of performance baseline.
  * `load`: Sustained target VU throughput testing.
  * `stress`: Ramp-up beyond capacity to detect degradation breakpoints.
  * `spike`: Instant surge in concurrent traffic to test elastic recovery.
* **Quality Gates**: Strict SLA thresholds evaluated at runtime (e.g. `p(95) < 500ms`, `errors < 1%`).
* **Artifact Reporting**: Automated HTML performance dashboards, raw metric summaries (`summary.json`), and GitHub Actions job summaries.

### 3. Test Case Management & Linear Sync (`squash-tm`)
* **Technology**: Squash TM (Java Spring Boot), PostgreSQL 15.
* **Automated Sync**: When developers or PMs label a ticket in Linear with `ready-for-tc`:
  1. Linear fires a webhook to `https://hooks.test-suites-poc.work.gd/webhooks/linear`.
  2. `omni-integration` validates the secret and parses preconditions, steps, and expected results.
  3. A new test case is created in Squash TM under project `omni-suites`.
  4. Mapping IDs are idempotently recorded in `omni_integration_db`.

### 4. AI-Driven Test Analytics & Dashboards (`report-portal`)
* **Technology**: ReportPortal v5.15 (12 microservices, PostgreSQL 18, RabbitMQ, OpenSearch, Python ML auto-analyzer).
* **Capabilities**:
  * Real-time test log streaming and video/screenshot artifact ingestion.
  * Flaky test identification and execution trends across releases.
  * Machine learning defect triage: automatically clusters failures into *Product Bug*, *Automation Bug*, *System Issue*, or *To Investigate*.

---

## 🤖 ChatOps & Developer Experience (Discord Bot)

Omni-Suites embeds QA triggers directly into developer team workflows through Discord Slash Commands:

| Command | Arguments | Action |
| :--- | :--- | :--- |
| **`/run-tests`** | `environment` (`dev`/`staging`), `suite` (`smoke`/`regression`/`api`/`order`/`inventory`), `browser` (`chromium`/`firefox`/`webkit`), `workers` | Dispatches Playwright test workflow on GitHub Actions. |
| **`/run-perf`** | `profile` (`smoke`/`load`/`stress`/`spike`), `service` (`all`/`order`/`inventory`/`notification`/`checkout`), `duration`, `vus` | Dispatches k6 performance suite on GitHub Actions. |

Upon completion, the bot posts rich status embeds to the `#test-releases` channel containing pass/fail metrics, duration, and direct links to GitHub Actions summaries and ReportPortal launches.

---

## ☁️ Cloud Infrastructure, Staging VM & Ingress

The staging cloud environment runs on a dedicated virtual machine (`/opt/omni-infra`) managed with modular Docker Compose stacks:

```text
/opt/omni-infra/
├── traefik/         # Ingress reverse proxy, TLS termination, edge network
├── apps/            # Microservices, omni-client, databases (Postgres, MySQL, Mongo)
├── squash-tm/       # Squash TM application & dedicated PostgreSQL database
└── report-portal/   # ReportPortal v5.15 distributed analytics cluster
```

### Network Isolation Policy
* **`edge` (External Bridge)**: Created by Traefik. Connects public-facing HTTP backends (`omni-client`, `order-service`, `inventory-service`, `notification-service`, `omni-integration`, `squash-tm`, `report-portal-gateway`).
* **`apps-internal` (Private)**: Completely isolates databases (`postgres-db`, `mysql-db`, `mongo-db`) and direct inter-service RPC calls.
* **`squash-internal` (Private)**: Dedicated link between `squash-tm` and its PostgreSQL instance.
* **`reportportal` (Private)**: Internal mesh between ReportPortal microservices, PostgreSQL 18, RabbitMQ, and OpenSearch.

---

## 🌐 Master Public Routing Directory

All services are publicly resolved via HTTPS under the `test-suites-poc.work.gd` domain:

| Domain Name | Target Component | Service Port | Protocol | Description |
| :--- | :--- | :--- | :---: | :--- |
| `frontend-svc.test-suites-poc.work.gd` | `omni-client` | `80` | HTTPS | Customer Frontend Web UI |
| `order-svc.test-suites-poc.work.gd` | `order-service` | `3000` | HTTPS | Order Microservice REST API |
| `inventory-svc.test-suites-poc.work.gd` | `inventory-service` | `3001` | HTTPS | Inventory Microservice REST API |
| `notification-svc.test-suites-poc.work.gd` | `notification-service`| `3002` | HTTPS | Notification Microservice REST API |
| `hooks.test-suites-poc.work.gd` | `omni-integration` | `3003` | HTTPS | Linear Webhooks & Discord Bot API |
| `squash-tcms.test-suites-poc.work.gd/squash` | `squash-tm` | `8080` | HTTPS | Squash TM Test Case Management |
| `report-portal.test-suites-poc.work.gd` | `report-portal-gateway`| `8080` | HTTPS | ReportPortal Analytics & Dashboards |

---

## 🔐 Security, Networking & Secrets Management

1. **Firewall & Ingress Surface**:
   * Only ports `22` (SSH), `80` (HTTP challenge / redirect), and `443` (HTTPS) are exposed on the host VM.
   * **Zero Host Database Ports**: No database ports (`5432`, `3306`, `27017`, `9200`, `5672`) are published to the host or internet.
2. **Automated SSL/TLS**:
   * Traefik v3 manages Let's Encrypt certificates using ACME HTTP-01 challenges, storing keys securely in the `letsencrypt` Docker volume (`/letsencrypt/acme.json`).
3. **Cryptographic Validation**:
   * Linear webhooks verify HMAC signatures against `LINEAR_WEBHOOK_SECRET`.
   * Discord interactions verify cryptographic Ed25519 signatures against `DISCORD_PUBLIC_KEY`.
4. **Environment Isolation**:
   * All `.env` files are strictly excluded from version control via `.gitignore`.
   * GitHub Actions secrets and variables inject credentials securely into CI/CD runners.

---

## 💻 Local Development & Contribution Workflow

### Cloning the Workspace
```bash
# Clone the central workspace
git clone https://github.com/omni-suites/.github.git omni-suites
cd omni-suites

# Clone individual repositories into app/ and quality suites
git clone https://github.com/omni-suites/omni-client.git app/omni-client
git clone https://github.com/omni-suites/order-service.git app/order-service
git clone https://github.com/omni-suites/inventory-service.git app/inventory-service
git clone https://github.com/omni-suites/notification-service.git app/notification-service
git clone https://github.com/omni-suites/omni-integration.git app/omni-integration
git clone https://github.com/omni-suites/test-suites.git test-suites
git clone https://github.com/omni-suites/perf-suites.git perf-suites
git clone https://github.com/omni-suites/infra.git infra
```

### Running Test Automation Locally

#### Playwright E2E & API Suites
```bash
cd test-suites
cp .env.sample .env
npm install
npx playwright test --project=chromium
```

#### Grafana k6 Performance Suites
```bash
cd perf-suites
cp .env.sample .env
npm install
npm run build
npm run test:smoke
```

---

<div align="center">
  <sub>Built with ❤️ by the <b>Omni-Suites</b> Engineering & Quality Assurance Team.</sub> <br>
  <sub>An Architecture and Concept by <a href="https://github.com/moshdev2213">moshdev2213</a>.</sub>
</div>