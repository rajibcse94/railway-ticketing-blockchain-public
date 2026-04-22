# A Pathway to Cloud-Based Blockchain-Enabled Ticketing System for National Railway of Bangladesh
### Blockchain-Powered | Full-Stack | Real-Time | QR Tickets | Cloud-Ready

A production-grade railway ticketing platform built for Bangladesh Railway. Every ticket booking, cancellation, user registration, and login is permanently recorded on a real Proof-of-Work blockchain as a mined block. Passengers search trains, select seats, pay, and receive QR-coded digital tickets. Admins manage the entire railway operation in real-time. Inspectors verify tickets on board.

---

## Table of Contents

1. [System Overview](#1-system-overview)
2. [System Architecture — Layered View](#2-system-architecture--layered-view)
3. [Architecture — Data Flow Diagram](#3-architecture--data-flow-diagram)
4. [Architecture — Component Diagram](#4-architecture--component-diagram)
5. [Technology Stack](#5-technology-stack)
6. [Fresh Ubuntu 22.04 Setup — Full Guide](#6-fresh-ubuntu-2204-setup--full-guide)
7. [Environment Variables](#7-environment-variables)
8. [Start / Stop the System](#8-start--stop-the-system)
9. [Seed the Database](#9-seed-the-database)
10. [Default Logins & Ports](#10-default-logins--ports)
11. [User Roles & Permissions](#11-user-roles--permissions)
12. [How the System Works — End to End](#12-how-the-system-works--end-to-end)
13. [Ticket Booking Flowchart](#13-ticket-booking-flowchart)
14. [Refund & Cancellation Policy](#14-refund--cancellation-policy)
15. [Real-Time Updates — WebSocket](#15-real-time-updates--websocket)
16. [Notification System](#16-notification-system)
17. [Blockchain — How It Actually Works](#17-blockchain--how-it-actually-works)
18. [Security Features](#18-security-features)
19. [API Reference](#19-api-reference)
20. [Seed Scripts](#20-seed-scripts)
21. [Hyperledger Fabric (Optional)](#21-hyperledger-fabric-optional)
22. [Production Deployment](#22-production-deployment)
23. [Project Structure](#23-project-structure)

---

## 1. System Overview

| Component | Description |
|---|---|
| **Passenger App** | Search trains, book seats, cancel tickets, view QR, receive notifications |
| **Inspector App** | Scan QR codes, verify tickets, mark as USED, flag suspicious tickets |
| **Admin Dashboard** | Full system management, real-time stats, blockchain ledger, notifications |
| **Backend API** | Node.js + Express, JWT auth, RBAC, Joi validation, Socket.IO real-time |
| **MongoDB (Off-chain)** | Tickets, users, trains, routes, stations, OTPs, notifications, audit logs |
| **Python Blockchain (On-chain)** | Every booking, cancellation, login, registration mined as PoW block |
| **Hyperledger Fabric** | Enterprise permissioned blockchain layer (optional, configurable) |
| **Email Service** | Gmail SMTP — HTML notifications on train cancel / reschedule |
| **WebSocket** | Socket.IO — admin dashboard updates instantly on any passenger action |

---

## 2. System Architecture — Layered View

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                        LAYER 1 — CLIENT LAYER                               ║
║                     (React 18  ·  Nginx  ·  Docker)                         ║
╠══════════════════╦══════════════════════╦════════════════════════════════════╣
║  Passenger App   ║   Inspector App      ║   Admin Dashboard                  ║
║  Port :3001      ║   Port :3002         ║   Port :3003                       ║
║                  ║                      ║                                    ║
║  • Register      ║  • QR Scan           ║  • Dashboard Overview              ║
║  • Login / MFA   ║  • Ticket Verify     ║  • Train Management                ║
║  • Search Trains ║  • Mark Used         ║  • Ticket Management               ║
║  • Select Seats  ║  • Flag Suspicious   ║  • User Management                 ║
║  • Book Ticket   ║                      ║  • Revenue Reports                 ║
║  • Cancel Ticket ║                      ║  • Blockchain Ledger               ║
║  • QR Ticket     ║                      ║  • Notifications                   ║
║  • Notifications ║                      ║  • Real-time Socket.IO             ║
║  • Wallet View   ║                      ║  • Refund Policy                   ║
╚══════════════════╩══════════════════════╩════════════════════════════════════╝
         │  HTTP REST / JSON                         │ Socket.IO (WebSocket)
         ▼                                           ▼
╔══════════════════════════════════════════════════════════════════════════════╗
║                      LAYER 2 — API GATEWAY LAYER                            ║
║                    Node.js · Express · Port :3000                           ║
╠══════════════════════════════════════════════════════════════════════════════╣
║  • CORS Whitelist (allowed origins only)                                    ║
║  • Helmet (X-Frame, XSS, HSTS, CSP security headers)                       ║
║  • Morgan (HTTP request logging)                                            ║
║  • Socket.IO Server (real-time event broadcasting)                         ║
║  • Route Mounting:                                                          ║
║      /api/auth          → authRoutes                                        ║
║      /api/passenger     → passengerRoutes                                   ║
║      /api/inspector     → inspectorRoutes                                   ║
║      /api/admin         → adminRoutes                                       ║
║      /api/superadmin    → superAdminRoutes                                  ║
║      /api/stakeholders  → stakeholderRoutes                                 ║
╚══════════════════════════════════════════════════════════════════════════════╝
         │
         ▼
╔══════════════════════════════════════════════════════════════════════════════╗
║                     LAYER 3 — MIDDLEWARE LAYER                              ║
╠════════════════════╦═══════════════════╦══════════════════╦═════════════════╣
║  auth.js           ║  authorize.js     ║  validate.js     ║ errorHandler.js ║
║                    ║                   ║                  ║                 ║
║  • Verify JWT      ║  • RBAC check     ║  • Joi schema    ║  • Catch errors ║
║  • Decode user     ║  • Match roles:   ║  • Sanitize body ║  • Format resp  ║
║  • Attach req.user ║    passenger      ║  • Reject unkn.  ║  • Log errors   ║
║                    ║    inspector      ║    fields        ║                 ║
║                    ║    admin          ║                  ║                 ║
║                    ║    superadmin     ║                  ║                 ║
╚════════════════════╩═══════════════════╩══════════════════╩═════════════════╝
         │
         ▼
╔══════════════════════════════════════════════════════════════════════════════╗
║                    LAYER 4 — BUSINESS LOGIC LAYER                           ║
║                           (Controllers)                                     ║
╠════════════════════╦══════════════════╦══════════════════╦══════════════════╣
║ authController     ║passengerCtrl     ║ adminController  ║ inspectorCtrl   ║
║                    ║                  ║                  ║                  ║
║ • register         ║ • searchTrains   ║ • manageTrains   ║ • verifyTicket  ║
║ • login            ║ • getSeats       ║ • manageRoutes   ║ • markUsed      ║
║ • MFA setup/verify ║ • bookTicket     ║ • getTickets     ║ • flagTicket    ║
║ • refreshToken     ║ • cancelTicket   ║ • cancelTrain    ║                 ║
║ • OTP send/verify  ║ • getMyTickets   ║ • notifyUsers    ║superAdminCtrl   ║
║                    ║ • QR generate    ║ • revenueReport  ║                 ║
║ Emits:             ║ • waitlist       ║ • refundPolicy   ║ • createUser    ║
║  user:registered   ║ • notifications  ║                  ║ • editUser      ║
║  user:login        ║                  ║ Emits:           ║ • deleteUser    ║
║                    ║ Emits:           ║  (train events)  ║ • revenueView   ║
║                    ║  ticket:booked   ║                  ║                 ║
║                    ║  ticket:cancelled║                  ║                 ║
╚════════════════════╩══════════════════╩══════════════════╩══════════════════╝
         │
         ▼
╔══════════════════════════════════════════════════════════════════════════════╗
║                     LAYER 5 — SERVICE LAYER                                 ║
╠══════════════════════════╦═══════════════════════╦═══════════════════════════╣
║  blockchainService.js    ║  fabricService.js     ║  encryption.js           ║
║                          ║                       ║                          ║
║  • createWallet          ║  • submitTransaction  ║  • AES-256-GCM encrypt   ║
║  • payWithBlockchain     ║  • evaluateTransaction║  • AES-256-GCM decrypt   ║
║  • processRefund         ║  • queryTicketHash    ║  • Random IV per value   ║
║  • recordTicketOnChain   ║                       ║                          ║
║  • recordCancellation    ║  refundCalculator.js  ║  captureLocation.js      ║
║    OnChain               ║                       ║                          ║
║  • recordUserRegistration║  • Calculate refund % ║  • IP → city/country     ║
║  • recordUserLogin       ║  • Apply policy       ║  • Store per-login       ║
║  • getTransactionHistory ║  • Check hours until  ║  • GPS coordinates       ║
║  • getBlockchainBalance  ║    departure          ║                          ║
╚══════════════════════════╩═══════════════════════╩═══════════════════════════╝
         │                              │
         ▼                              ▼
╔═════════════════════════╗   ╔════════════════════════════════════════════════╗
║  LAYER 6 — DATA LAYER   ║   ║     LAYER 7 — BLOCKCHAIN LAYER                ║
║  MongoDB  Port :27017   ║   ╠════════════════════╦═══════════════════════════╣
╠═════════════════════════╣   ║ Python PoW Chain   ║  Hyperledger Fabric       ║
║  Collections:           ║   ║ Port :5001         ║  (Optional)               ║
║  • users                ║   ║                    ║                           ║
║  • tickets              ║   ║ • Mine blocks      ║  • CA       :7054         ║
║  • trains               ║   ║ • SHA-256 PoW      ║  • Orderers :7050         ║
║  • routes               ║   ║ • Wallet CRUD      ║  • Peer0    :7051         ║
║  • stations             ║   ║ • TX history       ║  • Peer1    :8051         ║
║  • notifications        ║   ║ • Block explorer   ║  • Channel: railwaychan   ║
║  • otps                 ║   ║                    ║  • Chaincode: ticket (Go) ║
║  • waitlists            ║   ║ Data files:        ║                           ║
║  • refundpolicies       ║   ║ blockchain.json    ║  Enabled via:             ║
║  • stakeholders         ║   ║ users.json         ║  USE_REAL_FABRIC=true     ║
║  • profileupdatelogs    ║   ║ wallets.json       ║                           ║
╚═════════════════════════╝   ║ mempool.json       ║                           ║
                              ╚════════════════════╩═══════════════════════════╝
```

---

## 3. Architecture — Data Flow Diagram

```
  PASSENGER ACTION                     ADMIN DASHBOARD
  ─────────────                        ───────────────
  [Book Ticket]                        [Real-time view]
       │                                      ▲
       │  POST /api/passenger/tickets/book    │  Socket.IO
       ▼                                      │  ticket:booked event
  ┌─────────────────────────────────┐         │
  │   Node.js Backend  :3000        │─────────┘
  │                                 │
  │  1. Validate JWT + role         │
  │  2. Joi validate request body   │
  │  3. Check seat availability     │
  │     (query Ticket collection)   │
  │  4. Deduct blockchain wallet    │──────► Python Blockchain :5001
  │  5. Save Ticket to MongoDB      │──────► MongoDB :27017
  │  6. Emit ticket:booked socket   │──────► Admin Dashboard (instant)
  │  7. Mine block on blockchain    │──────► Python Blockchain :5001
  │     (non-blocking)              │         (TICKET_BOOKING tx)
  │  8. Submit to Fabric (optional) │──────► Hyperledger Fabric :7051
  │  9. Return ticket + QR code     │
  └─────────────────────────────────┘
       │
       ▼
  [QR Code displayed to passenger]

  ──────────────────────────────────────────────────────────────────

  INSPECTOR ACTION
  ─────────────────
  [Scan QR Code]
       │
       │  POST /api/inspector/tickets/verify
       ▼
  ┌─────────────────────────────────┐
  │   Node.js Backend  :3000        │
  │                                 │
  │  1. Decode QR JSON payload      │
  │  2. Find ticket in MongoDB      │──────► MongoDB :27017
  │  3. Verify SHA-256 hash match   │
  │  4. Check status = ISSUED       │
  │  5. Check correct train + date  │
  │  6. Return: VALID / INVALID     │
  └─────────────────────────────────┘
       │
       ▼
  [Inspector marks USED → ticket status permanently updated]

  ──────────────────────────────────────────────────────────────────

  BLOCKCHAIN MINING FLOW (every booking / cancellation / login)
  ─────────────────────────────────────────────────────────────
  Backend → POST /api/record/ticket  →  Python Blockchain
                                              │
                                    Build transaction dict
                                    {type: TICKET_BOOKING,
                                     txid: SHA256(data+ts),
                                     ticket_number, passenger,
                                     train, route, date, amount,
                                     data_hash}
                                              │
                                    Add to mempool
                                              │
                                    mine_next_block()
                                              │
                                    Increment nonce until
                                    SHA256(header) starts with 7482
                                              │
                                    Append block to blockchain.json
                                              │
                                    Return {blockHeight, blockHash}
                                              │
                                    Backend logs:
                                    "[BLOCKCHAIN] Ticket TICKET-xxx
                                     → block #N hash=7482abc..."
```

---

## 4. Architecture — Component Diagram

```
┌─────────────────── DOCKER NETWORK: railway-network ───────────────────────┐
│                                                                            │
│  ┌──────────────────┐   ┌──────────────────┐   ┌──────────────────────┐  │
│  │  passenger-app   │   │  inspector-app   │   │   admin-dashboard    │  │
│  │  React 18        │   │  React 18        │   │   React 18           │  │
│  │  Nginx           │   │  Nginx           │   │   Nginx + Socket.IO  │  │
│  │  :3001           │   │  :3002           │   │   :3003              │  │
│  └────────┬─────────┘   └────────┬─────────┘   └──────────┬───────────┘  │
│           │                      │                         │              │
│           └──────────────────────┴─────────────────────────┘              │
│                                  │  REST + WebSocket                      │
│                                  ▼                                        │
│  ┌───────────────────────────────────────────────────────────────────┐   │
│  │              railway-backend   :3000                               │   │
│  │  Node.js · Express · Socket.IO · JWT · Bcrypt · Helmet            │   │
│  └──────────┬────────────────────────────────────────┬───────────────┘   │
│             │                                         │                   │
│             ▼                                         ▼                   │
│  ┌──────────────────────┐              ┌─────────────────────────────┐   │
│  │  mongodb   :27017    │              │  blockchain   :5001         │   │
│  │  MongoDB 5           │              │  Python Flask + PoW         │   │
│  │  Volume:             │              │  Volume: blockchain-data     │   │
│  │  mongodb-data        │              └──────────────────────────── ┘   │
│  └──────────────────────┘                                                 │
│                                                                            │
│  ── Hyperledger Fabric (Optional) ───────────────────────────────────    │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐      │
│  │orderer0  │ │orderer1  │ │orderer2  │ │peer0     │ │peer1     │      │
│  │:7050     │ │:7052     │ │:7053     │ │:7051     │ │:8051     │      │
│  └──────────┘ └──────────┘ └──────────┘ └────┬─────┘ └────┬─────┘      │
│                                               ▼            ▼             │
│                                         ┌──────────┐ ┌──────────┐       │
│                                         │couchdb0  │ │couchdb1  │       │
│                                         │:5984     │ │:5984     │       │
│                                         └──────────┘ └──────────┘       │
└────────────────────────────────────────────────────────────────────────── ┘

  BLOCKCHAIN BLOCK STRUCTURE
  ──────────────────────────
  ┌──────────────────────────────────────────────┐
  │  Block #N                                    │
  │  ┌────────────────────────────────────────┐  │
  │  │ block_header                           │  │
  │  │   prevBlockHash : "7482a3b8f..."       │  │
  │  │   merkleRoot    : "9c338e45..."        │  │
  │  │   timestamp     : 1776843335           │  │
  │  │   nonce         : 147521               │  │
  │  │   blockHash     : "74824cd9..."  ◄ PoW │  │
  │  └────────────────────────────────────────┘  │
  │  txs: [                                      │
  │    { type: "coinbase",       txid: "..." }   │
  │    { type: "TICKET_BOOKING", txid: "...",    │
  │      ticket_number, passenger_name,          │
  │      train_number, from_station,             │
  │      to_station, journey_date, amount,       │
  │      data_hash }                             │
  │  ]                                           │
  └──────────────────────────────────────────────┘
         │ prevBlockHash links to:
         ▼
  ┌──────────────────────────────────────────────┐
  │  Block #N-1                                  │
  │   blockHash: "7482a3b8f..."                  │
  └──────────────────────────────────────────────┘

  TRANSACTION TYPES RECORDED AS BLOCKS
  ─────────────────────────────────────
  USER_REGISTRATION  → new passenger registers
  USER_LOGIN         → any user logs in
  TICKET_BOOKING     → passenger books a ticket
  TICKET_CANCELLATION→ passenger cancels a ticket
  payment            → wallet payment transaction
```

---

## 5. Technology Stack

### Backend — Node.js API Server

| Package | Version | Purpose |
|---|---|---|
| express | 4.18 | HTTP framework |
| socket.io | 4.8 | Real-time WebSocket server |
| mongoose | 7.x | MongoDB ODM |
| jsonwebtoken | 9.x | Access token (15 min) + refresh token (7 days) |
| bcryptjs | 2.4 | Password hashing (cost factor 10) |
| speakeasy | 2.0 | TOTP — Google Authenticator MFA |
| nodemailer | 8.x | Email via Gmail SMTP |
| joi | 17.x | Request body schema validation |
| helmet | 7.x | HTTP security headers |
| qrcode | 1.5 | QR ticket image generation |
| geoip-lite | 1.4 | IP → city/country for login tracking |
| cors | 2.8 | Origin whitelist |
| axios | 1.14 | Internal calls to blockchain service |
| morgan | 1.10 | HTTP request logging |
| dotenv | 16.x | Environment configuration |

### Frontend — React 18 (all three apps)

| Package | Version | Purpose |
|---|---|---|
| react | 18.2 | UI framework |
| react-router-dom | 6.10 | Client-side routing |
| axios | 1.3 | API calls + 401 auto-refresh interceptor |
| socket.io-client | 4.8 | Real-time WebSocket client (admin dashboard) |
| recharts | 2.5 | Charts — pie, bar, line (admin dashboard) |
| qrcode.react | 3.1 | QR code display (passenger + inspector) |

### Python Blockchain Service

| Package | Purpose |
|---|---|
| Flask | REST API framework |
| flask-cors | Cross-origin resource sharing |
| bcrypt | Wallet password hashing |
| PyJWT | Blockchain authentication tokens |
| pycryptodome | Elliptic curve cryptography |
| hashlib (stdlib) | SHA-256 Proof-of-Work mining |

### Infrastructure & DevOps

| Tool | Purpose |
|---|---|
| Docker + Docker Compose | All services containerised and orchestrated |
| MongoDB 5 | Primary off-chain data store |
| Nginx | Static frontend serving (all 3 apps) |
| Hyperledger Fabric 2.x | Enterprise permissioned blockchain (optional) |
| CouchDB 3.1 | Hyperledger Fabric world-state database |
| Ubuntu 22.04 LTS | Recommended host OS |

---

## 6. Fresh Ubuntu 22.04 Setup — Full Guide

### Step 1 — System update & base tools

```bash
sudo apt update && sudo apt upgrade -y

sudo apt install -y \
  curl wget git gnupg2 ca-certificates \
  lsb-release software-properties-common \
  apt-transport-https build-essential
```

### Step 2 — Install Docker Engine

```bash
# Add Docker GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Add Docker repo
echo \
  "deb [arch=$(dpkg --print-architecture) \
  signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Run Docker without sudo
sudo usermod -aG docker $USER
newgrp docker

# Verify
docker --version
docker compose version
```

### Step 3 — Install Node.js 18

```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

node --version   # v18.x.x
npm --version
```

### Step 4 — Install Python 3

```bash
sudo apt install -y python3 python3-pip python3-venv

python3 --version   # 3.10+
pip3 --version
```

### Step 5 — Clone the project

```bash
git clone https://github.com/rajibcse94/railway-ticketing-blockchain.git
cd railway-ticketing-blockchain
```

### Step 6 — Create environment file

```bash
cp .env.example .env
nano .env
```

Fill in all values — see [Section 7](#7-environment-variables).

### Step 7 — Build and start everything

```bash
docker compose up --build -d
```

Builds and starts: MongoDB, Python blockchain, Node.js backend, passenger app, inspector app, admin dashboard, Hyperledger Fabric nodes.

### Step 8 — Seed the database

```bash
# 1. Create super admin account
docker exec railway-backend node scripts/seed-db.js

# 2. Seed 50+ Bangladesh Railway stations
docker exec railway-backend node scripts/seedStations.js

# 3. Seed 8 real Bangladesh Railway routes
docker exec railway-backend node scripts/seedRoutes.js

# 4. Seed 14 trains with carriages + 30-day schedules
docker exec railway-backend node scripts/seedTrains.js
```

### Step 9 — Verify

```bash
docker ps                              # all containers running
curl http://localhost:3000/health      # returns: OK
curl http://localhost:5001/health      # returns: {"status":"ok"}
```

---

## 7. Environment Variables

Create **two identical files**: `.env` (project root) and `backend/.env`

```env
# ── Server ──────────────────────────────────────────────────
PORT=3000
NODE_ENV=production

# ── JWT ─────────────────────────────────────────────────────
# Generate: node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"
JWT_SECRET=replace_with_64_char_hex
JWT_REFRESH_SECRET=replace_with_another_64_char_hex
JWT_EXPIRES_IN=15m

# ── MongoDB ──────────────────────────────────────────────────
MONGO_URI=mongodb://admin:admin123@mongodb:27017/railway-ticketing?authSource=admin
MONGO_ROOT_USERNAME=admin
MONGO_ROOT_PASSWORD=admin123

# ── AES-256-GCM Data Encryption ─────────────────────────────
# Generate: node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
# Must be exactly 64 hex characters. Never change after data is stored.
ENCRYPTION_KEY=replace_with_64_char_hex

# ── Python Blockchain ────────────────────────────────────────
BLOCKCHAIN_URL=http://blockchain:5001
USE_MOCK_BLOCKCHAIN=true

# ── Hyperledger Fabric (optional) ────────────────────────────
USE_REAL_FABRIC=false
FABRIC_CONNECTION_PROFILE=/app/config/connection-profile.json
FABRIC_WALLET_PATH=/app/wallet
FABRIC_CHANNEL=railwaychannel
FABRIC_CHAINCODE=ticket

# ── Gmail SMTP (for train notifications) ─────────────────────
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USER=your_gmail@gmail.com
# Gmail → Account → Security → 2-Step Verification → App Passwords
EMAIL_PASS=xxxx_xxxx_xxxx_xxxx

# ── CORS ─────────────────────────────────────────────────────
CORS_ORIGINS=http://localhost:3001,http://localhost:3002,http://localhost:3003

# ── CouchDB (Hyperledger Fabric) ─────────────────────────────
COUCHDB_USER=admin
COUCHDB_PASSWORD=adminpw
```

---

## 8. Start / Stop the System

```bash
# Start all services
docker compose up -d

# Stop all services (data preserved)
docker compose down

# Full reset — deletes ALL data including MongoDB and blockchain
docker compose down -v

# Rebuild after code changes
docker compose up --build -d

# Rebuild a specific service only
docker compose up -d --build backend
docker compose up -d --build admin-dashboard
docker compose up -d --build blockchain

# Restart one service
docker compose restart backend
docker compose restart blockchain

# Live logs
docker logs railway-backend -f
docker logs blockchain -f
docker logs admin-dashboard -f
docker logs mongodb -f
```

---

## 9. Seed the Database

```bash
# Super admin: superadmin@railway.com / Admin@123
docker exec railway-backend node scripts/seed-db.js

# 50+ stations with GPS, division, zone
docker exec railway-backend node scripts/seedStations.js

# 8 Bangladesh Railway routes
docker exec railway-backend node scripts/seedRoutes.js

# 14 trains, carriages, 30-day schedules
docker exec railway-backend node scripts/seedTrains.js
```

All seed scripts clear their collection before inserting — safe to re-run.

### Trains Seeded

| No. | Name | Route |
|---|---|---|
| 711 / 712 | Subarna Express | Dhaka ↔ Chittagong |
| 741 / 742 | Turna Express | Dhaka ↔ Chittagong (night) |
| 767 / 768 | Parabat Express | Dhaka ↔ Sylhet |
| 725 / 726 | Sundarban Express | Dhaka ↔ Khulna |
| 759 / 760 | Padma Express | Dhaka ↔ Rajshahi |
| 745 / 746 | Jamuna Express | Dhaka ↔ Mymensingh |
| 1 / 2 | Cox's Bazar Express | Chittagong ↔ Cox's Bazar |

### Carriage Classes

| Class | Seats | Price / Seat |
|---|---|---|
| Shovon | 80 | ৳ 120 |
| Shovon Chair | 70 | ৳ 175 |
| Snigdha (AC) | 48 | ৳ 345 |
| First Class Seat | 56 | ৳ 280 |
| First Class Berth | 32 | ৳ 560 |
| AC Berth | 32 | ৳ 850 |
| AC Seat | 56 | ৳ 490 |

---

## 10. Default Logins & Ports

### Service Ports

| Service | URL | Description |
|---|---|---|
| Passenger App | http://localhost:3001 | Public booking portal |
| Inspector App | http://localhost:3002 | On-board ticket verification |
| Admin Dashboard | http://localhost:3003 | System management panel |
| Backend API | http://localhost:3000/api | REST API + WebSocket |
| Blockchain Node | http://localhost:5001 | Python PoW blockchain |
| Block Explorer | http://localhost:5001/blocks | Visual block explorer |
| Fabric CA | http://localhost:7054 | Certificate authority |
| Fabric Orderer | http://localhost:7050 | Transaction ordering |
| Fabric Peer 0 | http://localhost:7051 | Primary peer node |
| Fabric Peer 1 | http://localhost:8051 | Secondary peer node |

### Default Credentials

| Role | Email | Password |
|---|---|---|
| Super Admin | superadmin@railway.com | Admin@123 |
| Admin | Created by super admin | Set by super admin |
| Inspector | Created by admin | Set by admin |
| Passenger | Register via app | Set during registration |

---

## 11. User Roles & Permissions

```
superadmin
  ├── All admin capabilities
  ├── Create / edit / delete users of any role
  ├── View decrypted sensitive fields (NID, DOB, mobile)
  ├── Reset MFA for any user
  └── Change any user's role or password

admin
  ├── Manage trains, routes, stations, carriages, schedules
  ├── Cancel or reschedule trains → triggers email + in-app notifications
  ├── View all tickets with filter by status / date / train
  ├── View all passengers with full login history
  ├── View OTP records
  ├── Configure refund policy thresholds
  ├── View all sent notifications
  ├── View real-time blockchain ledger and block explorer
  └── Manage API stakeholders

inspector
  ├── Scan passenger QR code
  ├── Verify ticket validity (train, date, status, hash)
  ├── Mark ticket as USED after boarding
  └── Flag suspicious or tampered ticket

stationoperator
  └── View train and station information

passenger
  ├── Register with email OTP verification
  ├── Optional: enable Google Authenticator MFA
  ├── Search trains from any station to any station
  │     (supports intermediate stops: Dhaka→Comilla on Dhaka→Chittagong train)
  ├── Select carriage class and specific seat numbers
  ├── Book ticket — direct or via blockchain wallet payment
  ├── View booked tickets with QR code
  ├── Verify ticket SHA-256 integrity hash
  ├── Cancel ticket — view refund estimate, confirm with password
  ├── Join / leave waiting list
  ├── Update profile (email OTP required)
  └── Receive real-time in-app + email notifications
```

---

## 12. How the System Works — End to End

### Passenger Journey

```
REGISTER
  Enter: name, email, mobile, NID/passport, date of birth, gender, password
  System sends 6-digit OTP to email (expires in 10 minutes)
  Passenger enters OTP → account created
  Sensitive fields (NID, DOB, mobile) encrypted with AES-256-GCM
  Blockchain wallet created automatically with ৳20,000 starting balance
  USER_REGISTRATION transaction mined as real PoW block
  Optional: enable MFA (scan QR with Google Authenticator)

LOGIN
  Enter email + password
  If MFA enabled → enter 6-digit TOTP from Authenticator app
  USER_LOGIN transaction mined as real PoW block
  Receive: JWT access token (15 min) + refresh token (7 days)
  Axios interceptor silently refreshes access token on 401 — no logout

SEARCH TRAINS
  Enter: From station → To station → Date
  System queries all train stops (not just origin/destination)
  Example: Dhaka→Comilla returns Subarna Express (Dhaka→Chittagong)
           showing actual departure from Dhaka and arrival at Comilla
  Results ordered by departure time

SELECT CARRIAGE
  Choose class: Shovon / Shovon Chair / Snigdha / First Class / AC Berth
  Seat layout loads — green = available, red = booked for that date

SELECT SEATS
  Click 1–4 seats (system enforces max 4 active booked seats per passenger)
  Total = seat count × price per seat for that class

BOOK TICKET
  Option A: Direct booking (no wallet needed)
  Option B: Pay from blockchain wallet (enter wallet password)
             → balance deducted on blockchain service
  Ticket saved in MongoDB with:
    • Unique ticket number (TICKET-{timestamp})
    • SHA-256 integrity hash (ticketNumber + passengerId + trainId + journeyDate)
    • Passenger's actual boarding / alighting station
    • Carriage name, seat numbers, price, journey date
  TICKET_BOOKING transaction mined as PoW block:
    • Full ticket data in block transaction
    • Real Proof-of-Work — hash starts with 7482
    • Permanently linked to chain — tamper-proof
  Admin dashboard updates INSTANTLY via Socket.IO

QR TICKET
  Passenger views QR code — contains full ticket JSON
  SHA-256 hash verifiable against MongoDB and blockchain record

ON BOARD
  Inspector scans passenger QR code
  System checks: ISSUED status, correct train, correct date, valid hash
  Inspector marks as USED → permanently updated in MongoDB

CANCELLATION
  Passenger selects ticket → sees exact refund amount before confirming
  Confirms with account password
  Refund calculated by hours remaining before departure
  If wallet was used → refund credited back to blockchain wallet
  TICKET_CANCELLATION transaction mined as PoW block
  Admin dashboard updates INSTANTLY via Socket.IO
```

---

## 13. Ticket Booking Flowchart

```
START
  │
  ▼
Enter: From Station → To Station → Date
  │
  ▼
Query trains where stops contain both stations (MongoDB $all)
Filter: fromStop index < toStop index (direction check)
  │
  ├── No trains found ──────────────────────────► "No trains available"
  │
  ▼
Display matching trains with departure/arrival times for the leg
  │
  ▼
Select train → select carriage → load seat layout (dynamic from tickets)
  │
  ▼
Select 1–4 seats
  │
  ▼
Check seats not in existing ISSUED/USED tickets for that train+date
  │
  ├── Seat taken ───────────────────────────────► "Seat already booked"
  │
  ▼
Check passenger has < 4 active (ISSUED/USED) tickets
  │
  ├── At limit ────────────────────────────────► "Max 4 seats per passenger"
  │
  ▼
Pay with blockchain wallet?
  │
  ├── YES → Enter wallet password
  │           ├── WRONG ──────────────────────► "Invalid blockchain password"
  │           └── OK → Deduct amount from wallet balance
  │
  └── NO → Proceed (post-paid / cash equivalent)
  │
  ▼
Save Ticket to MongoDB
  │
  ▼
Generate SHA-256 integrity hash
  │
  ▼
Emit Socket.IO event → Admin dashboard updates instantly
  │
  ▼
Mine TICKET_BOOKING block on Python blockchain (async, non-blocking)
  │   Build tx → add to mempool → mine_next_block()
  │   Nonce++ until SHA256(header) starts with 7482
  │   Append block to blockchain.json
  │
  ▼
[Hyperledger Fabric enabled?] → Submit to Fabric chaincode (optional)
  │
  ▼
Return QR-coded ticket to passenger
  │
END
```

---

## 14. Refund & Cancellation Policy

| Time Before Departure | Refund |
|---|---|
| More than 48 hours | 80 % |
| 24 to 48 hours | 50 % |
| 0 to 24 hours | 25 % |
| After departure | 0 % |

**Rules:**
- Passenger sees the exact refund amount before confirming cancellation
- Account password required to confirm cancellation
- If ticket was paid via blockchain wallet → refund credited back automatically
- Cancelled tickets cannot be reused or re-cancelled
- Policy thresholds are configurable by admin (Refund Policy section)
- If no policy is configured, the above defaults apply automatically
- Every cancellation is mined as a PoW block recording refund details

---

## 15. Real-Time Updates — WebSocket

Socket.IO is used to push live updates to the admin dashboard the moment a passenger action occurs — no polling, no page refresh.

```
Event Flow:
───────────
Passenger books ticket
  → Node.js emits: io.emit('ticket:booked', { ticketNumber, passengerName, ... })
  → Admin dashboard receives instantly
  → fetchTickets() + fetchChainData() called
  → Ticket counts, revenue, available seats update in < 1 second

Passenger cancels ticket
  → Node.js emits: io.emit('ticket:cancelled', { ticketNumber, refundAmount, ... })
  → Admin dashboard receives instantly
  → Cancelled count updates, available seats restored

New user registers
  → Node.js emits: io.emit('user:registered', { userId, name, email, role })
  → Total Users count updates instantly

User logs in
  → Node.js emits: io.emit('user:login', { userId, email, role })
  → Blockchain ledger refresh triggered
```

| Socket Event | Triggered By | Admin Dashboard Effect |
|---|---|---|
| `ticket:booked` | Ticket booking | Ticket counts, revenue, available seats |
| `ticket:cancelled` | Ticket cancellation | Cancelled count, seat availability |
| `user:registered` | New registration | Total users, passenger list |
| `user:login` | Any login | Blockchain ledger stats |

Polling every 30 seconds remains as fallback for any missed events.

---

## 16. Notification System

When admin cancels or reschedules a train:

```
Admin selects train → Cancel / Reschedule
  │
  ├── Cancel:     enter reason
  └── Reschedule: new date + new departure time + new arrival time
  │
  ▼
Backend finds all tickets on that train for that affected date
  │
  ▼
Finds all affected users (passengers, inspectors, station operators)
  │
  ▼
For each affected user:
  ┌─ Creates Notification record in MongoDB
  │    { type: TRAIN_CANCELLED | TRAIN_RESCHEDULED,
  │      title, message, trainNumber, trainName,
  │      isRead: false, emailSent: false }
  │
  └─ Sends HTML email via Gmail SMTP
       Red header  → train cancelled
       Blue header → train rescheduled
  │
  ▼
Passenger app polls /notifications every 15 seconds
  │
  ▼
Bell icon shows red badge with unread count
  │
  ▼
Toast notification appears bottom-right corner
  │
  ▼
Passenger clicks bell → panel opens → all marked as read

Admin Notifications tab:
  All sent notifications with:
  User Name | Email | Role | Type | Train | Message | Email Sent | Read | Time
```

---

## 17. Blockchain — How It Actually Works

### Python Custom Proof-of-Work Blockchain (Port 5001)

This is not a database pretending to be a blockchain. It is a real blockchain with real SHA-256 Proof-of-Work mining. Every hash starts with `7482`.

**Events recorded as blocks:**

| Event | Transaction Type | Data Stored |
|---|---|---|
| User registers | `USER_REGISTRATION` | userId, name, email, role, IP, timestamp |
| User logs in | `USER_LOGIN` | userId, name, email, role, IP, timestamp |
| Ticket booked | `TICKET_BOOKING` | ticket#, passenger, train, route, date, amount, integrity hash |
| Ticket cancelled | `TICKET_CANCELLATION` | ticket#, passenger, train, reason, refund amount, refund % |
| Wallet payment | `payment` | sender, receiver, amount |

**Block Structure:**
```json
{
  "height": 5,
  "block_header": {
    "version": 1,
    "prevBlockHash": "74823a8f...",
    "merkleRoot": "9c338e45...",
    "timestamp": 1776843335,
    "bits": "ffff001f",
    "nonce": 147521,
    "blockHash": "74824cd9..."
  },
  "tx_count": 2,
  "txs": [
    {
      "type": "coinbase",
      "txid": "85c3a159...",
      "amount": 5000000000
    },
    {
      "type": "TICKET_BOOKING",
      "txid": "e247a04f...",
      "ticket_number": "TICKET-1776843330",
      "passenger_id": "683abc...",
      "passenger_name": "Rajib Das",
      "train_number": "711",
      "train_name": "Subarna Express",
      "from_station": "Dhaka",
      "to_station": "Comilla",
      "journey_date": "2026-04-25",
      "carriage_name": "Shovon-1",
      "seat_numbers": [5, 6],
      "amount": 240.0,
      "data_hash": "a3f9b2...",
      "timestamp": 1776843335
    }
  ]
}
```

**Block hash starts with `7482` — that is real Proof-of-Work.**

**Mining flow for every event:**
```
Backend → POST /api/record/ticket (or /user-register, /user-login, /cancellation)
  → Python service builds transaction dict with SHA-256 txid
  → Adds tx to mempool.json
  → Calls mine_next_block()
  → Increments nonce until SHA256(header) starts with 7482
  → Appends block to blockchain.json
  → Returns { blockHeight, blockHash }
  → Backend logs: "[BLOCKCHAIN] Ticket TICKET-xxx → block #5 hash=74824cd9..."
```

**Blockchain API Endpoints:**

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/record/ticket` | Mine ticket booking as block |
| POST | `/api/record/cancellation` | Mine cancellation as block |
| POST | `/api/record/user-register` | Mine user registration as block |
| POST | `/api/record/user-login` | Mine user login as block |
| GET | `/api/blocks` | Full chain as JSON |
| GET | `/api/chain/stats` | Total blocks, bookings, cancellations, logins |
| POST | `/api/blockchain/register` | Create blockchain user |
| POST | `/api/blockchain/login` | Authenticate, get token |
| POST | `/api/blockchain/pay` | Deduct fare from wallet |
| POST | `/api/wallet` | Create wallet on registration |
| GET | `/api/wallet/<userId>` | Wallet balance + info |
| POST | `/api/wallet/refund` | Credit refund to wallet |
| GET | `/api/wallets` | All wallet balances |
| GET | `/blocks` | Block explorer web UI |

**Wallet System:**
- Every passenger gets a blockchain wallet on registration
- Starting balance: ৳ 20,000
- Wallet password stored bcrypt-hashed in `users.json`
- Balance tracked in `users.json` and `wallets.json`
- Transaction log in `tx_log.json`
- Refunds credited back automatically on cancellation

### Hyperledger Fabric (Optional — Port 7050–8053)

| Component | Role |
|---|---|
| Certificate Authority | Issues MSP identities (port 7054) |
| 3 Orderers (Raft consensus) | Transaction ordering |
| 2 Peers | Ledger + chaincode execution |
| 2 CouchDB | World-state rich query databases |
| Channel | `railwaychannel` |
| Chaincode | `ticket` (Go) |

Enable: set `USE_REAL_FABRIC=true` in `.env` and restart backend.

---

## 18. Security Features

### Authentication & Session Management

| Feature | Implementation |
|---|---|
| Access token | JWT, 15-minute expiry, signed with `JWT_SECRET` |
| Refresh token | JWT, 7-day expiry, stored in localStorage |
| Auto-refresh | Axios interceptor catches 401, refreshes silently, retries |
| MFA | TOTP via speakeasy — Google Authenticator compatible |
| MFA reset | Super admin can reset MFA for any user |

### Data Encryption at Rest

| Data Field | Method |
|---|---|
| Passwords | bcrypt, cost factor 10 — never stored plaintext |
| NID / Passport | AES-256-GCM, unique random IV per value |
| Date of birth | AES-256-GCM encrypted |
| Mobile number | AES-256-GCM encrypted |
| GCM auth tag | Tampered ciphertext rejected on decryption |

### Ticket Integrity

| Feature | Detail |
|---|---|
| Integrity hash | SHA-256 of: ticketNumber + passengerId + trainId + journeyDate |
| Stored in | MongoDB ticket record AND blockchain block transaction |
| Verifiable by | Passenger (QR screen) and Inspector (on scan) |

### Network & Access Control

| Feature | Implementation |
|---|---|
| Request validation | Joi schemas — types enforced, unknown fields stripped |
| Route protection | `auth` middleware (JWT verify) + `authorize(roles)` |
| HTTP security | Helmet: X-Frame-Options, XSS-Protection, HSTS, CSP |
| CORS | Whitelist only — `CORS_ORIGINS` env var |
| ETag disabled | No 304 cached responses — always fresh data |

### Audit & Tracking

| Feature | Implementation |
|---|---|
| Login location | IP → city/country via geoip-lite, stored per login |
| Login history | Last 20 logins stored per user |
| Profile changes | Before/after values logged in `profileupdatelogs` |
| OTP records | All OTPs stored and visible to admin |
| Blockchain immutability | All key events permanently recorded as PoW blocks |

---

## 19. API Reference

### Auth  `/api/auth`

| Method | Path | Auth | Description |
|---|---|---|---|
| POST | `/register/start` | None | Send OTP to email |
| POST | `/register/verify` | None | Verify OTP, activate account |
| POST | `/login` | None | Login — returns access + refresh tokens |
| POST | `/refresh` | None | Get new access token from refresh token |
| GET | `/me` | JWT | Current user profile |
| POST | `/mfa/setup` | JWT | Setup Google Authenticator |
| POST | `/mfa/verify` | JWT | Verify TOTP code |

### Passenger  `/api/passenger`

| Method | Path | Description |
|---|---|---|
| GET | `/trains/search?from=&to=&date=` | Search (supports intermediate stops) |
| GET | `/trains/:id` | Train details |
| GET | `/stations` | All stations |
| GET | `/seats?trainId=&date=&carriageName=` | Real-time seat availability |
| POST | `/tickets/book` | Book ticket |
| GET | `/tickets/my` | My tickets |
| POST | `/tickets/:id/cancel` | Cancel ticket (password required) |
| GET | `/tickets/:id/refund-estimate` | Refund preview before cancelling |
| GET | `/refund-policy` | Active refund policy |
| GET | `/tickets/:id/qr` | QR code image |
| GET | `/tickets/:id/verify` | Verify integrity hash |
| POST | `/waitlist/join` | Join waiting list |
| DELETE | `/waitlist/:id` | Leave waiting list |
| GET | `/waitlist/my` | My waiting list entries |
| GET | `/notifications` | My notifications |
| PUT | `/notifications/read` | Mark all read |
| POST | `/profile/request-otp` | OTP for profile update |
| PUT | `/profile/update` | Update profile |
| GET | `/wallet/transactions` | Blockchain wallet history |

### Admin  `/api/admin`

| Method | Path | Description |
|---|---|---|
| GET | `/users` | All users |
| GET | `/passengers` | All passengers + login history |
| GET | `/tickets` | All tickets (filter: status, date, train) |
| GET | `/trains` | All trains |
| POST | `/trains` | Add train |
| PUT | `/trains/:id` | Update train |
| DELETE | `/trains/:id` | Delete train |
| GET | `/routes` | All routes |
| POST | `/routes` | Create route |
| PUT | `/routes/:id` | Update route |
| DELETE | `/routes/:id` | Delete route |
| GET | `/stations` | All stations |
| POST | `/stations` | Add station |
| DELETE | `/stations/:id` | Delete station |
| POST | `/trains/:id/cancel-reschedule` | Cancel/reschedule + notify users |
| GET | `/notifications` | Admin's own notifications |
| GET | `/notifications/all` | All notifications (full admin view) |
| GET | `/revenue` | Revenue reports |
| GET | `/blockchain/chain` | Full blockchain (proxied from Python service) |
| GET | `/blockchain/stats` | Chain statistics |
| GET | `/blockchain/wallets` | All wallet balances |

### Super Admin  `/api/superadmin`

| Method | Path | Description |
|---|---|---|
| GET | `/users` | All users with decrypted sensitive fields |
| POST | `/users` | Create user (any role) |
| PUT | `/users/:id` | Edit user, set password, reset MFA, change role |
| DELETE | `/users/:id` | Delete user |
| GET | `/revenue` | Revenue reports |

### Inspector  `/api/inspector`

| Method | Path | Description |
|---|---|---|
| POST | `/tickets/verify` | Verify QR ticket |
| POST | `/tickets/:id/use` | Mark ticket as USED |
| POST | `/tickets/:id/flag` | Flag suspicious ticket |

---

## 20. Seed Scripts

```bash
# Run all in order (inside backend container)

docker exec railway-backend node scripts/seed-db.js
# Creates: superadmin@railway.com / Admin@123

docker exec railway-backend node scripts/seedStations.js
# Creates: 50+ stations with GPS, division, district, zone, neighbors

docker exec railway-backend node scripts/seedRoutes.js
# Creates 8 routes:
#   Dhaka–Chittagong Mainline       (10 stops)
#   Dhaka–Sylhet Mainline           (8 stops)
#   Dhaka–Khulna Mainline           (4 stops)
#   Dhaka–Rajshahi Mainline         (6 stops)
#   Dhaka–Mymensingh Line           (5 stops)
#   Chittagong–Cox's Bazar Line     (4 stops)
#   Dhaka–Rajshahi via Santahar     (5 stops)
#   Rajshahi–Rohanpur Line          (3 stops)

docker exec railway-backend node scripts/seedTrains.js
# Creates 14 trains (7 routes × up/down direction)
# Each train: all carriage classes + 30 days of schedules from today

# Utility scripts
docker exec railway-backend node scripts/encryptExistingUsers.js
# Re-encrypts sensitive user fields (run after ENCRYPTION_KEY change)

docker exec railway-backend node scripts/create-indexes.js
# Creates MongoDB indexes for query performance
```

---

## 21. Hyperledger Fabric (Optional)

```bash
# Generate certificates
cd blockchain
bash scripts/generate-certs.sh

# Start Fabric network
bash scripts/start-network.sh

# Create channel
bash scripts/create-channel.sh

# Deploy chaincode
bash scripts/deploy-chaincode.sh

# Enroll admin wallet identity
cd ..
node scripts/enroll-users.js

# Enable in backend
# Set USE_REAL_FABRIC=true in .env
docker compose restart backend
```

---

## 22. Production Deployment

### VPS Requirements

| Resource | Minimum | Recommended |
|---|---|---|
| CPU | 4 cores | 8 cores |
| RAM | 8 GB | 12 GB |
| Disk | 50 GB SSD | 250 GB SSD |
| OS | Ubuntu 22.04 LTS | Ubuntu 22.04 LTS |

### Generate Secure Keys

```bash
# Run three times — one output for each of:
#   JWT_SECRET, JWT_REFRESH_SECRET, ENCRYPTION_KEY
node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"
```

### Nginx Reverse Proxy

```nginx
server {
    listen 80;
    server_name yourdomain.com;

    # Backend API + WebSocket
    location /api/ {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # Socket.IO
    location /socket.io/ {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }

    # Passenger app
    location / {
        proxy_pass http://localhost:3001;
    }
}
```

### SSL Certificate

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d yourdomain.com
```

### Firewall

```bash
sudo ufw allow 22/tcp    # SSH
sudo ufw allow 80/tcp    # HTTP
sudo ufw allow 443/tcp   # HTTPS
sudo ufw deny 3000/tcp   # block direct backend
sudo ufw deny 5001/tcp   # block direct blockchain
sudo ufw deny 27017/tcp  # block direct MongoDB
sudo ufw enable
```

### Docker on Boot

```bash
sudo systemctl enable docker
```

### MongoDB Backup

```bash
# Backup
docker exec mongodb mongodump \
  --uri="mongodb://admin:PASSWORD@localhost:27017/railway-ticketing?authSource=admin" \
  --out=/backup/$(date +%Y%m%d)

# Restore
docker exec mongodb mongorestore \
  --uri="mongodb://admin:PASSWORD@localhost:27017/railway-ticketing?authSource=admin" \
  /backup/20260422
```

---

## 23. Project Structure

```
railway-ticketing-blockchain/
│
├── backend/
│   ├── controllers/
│   │   ├── authController.js          # register, login, refresh, MFA, OTP
│   │   ├── passengerController.js     # search, book, cancel, QR, notifications
│   │   ├── adminController.js         # train mgmt, cancel/reschedule, notifications
│   │   ├── inspectorController.js     # verify, mark used, flag
│   │   ├── mfaController.js           # MFA login flow
│   │   └── superAdminController.js    # user CRUD, revenue reports
│   ├── middleware/
│   │   ├── auth.js                    # JWT verify → attach req.user
│   │   ├── authorize.js               # RBAC role check
│   │   ├── validate.js                # Joi request body schemas
│   │   └── errorHandler.js            # Global error formatter
│   ├── models/
│   │   ├── User.js                    # users (AES-256 fields, login history)
│   │   ├── Ticket.js                  # tickets (SHA-256 hash, refund fields)
│   │   ├── Train.js                   # trains (carriages, schedules, stops)
│   │   ├── Route.js                   # routes (ordered stops)
│   │   ├── Station.js                 # stations (GPS, division, zone)
│   │   ├── Notification.js            # notifications (emailSent, isRead)
│   │   ├── OTP.js                     # email OTPs (expires in 10 min)
│   │   ├── WaitList.js                # waitlist entries per train/date/carriage
│   │   ├── RefundPolicy.js            # configurable refund thresholds
│   │   ├── Stakeholder.js             # API access stakeholders
│   │   └── ProfileUpdateLog.js        # profile change audit trail
│   ├── routes/
│   │   ├── authRoutes.js              # /api/auth
│   │   ├── passengerRoutes.js         # /api/passenger
│   │   ├── inspectorRoutes.js         # /api/inspector
│   │   ├── adminRoutes.js             # /api/admin (+ blockchain proxy)
│   │   ├── superAdminRoutes.js        # /api/superadmin
│   │   └── stakeholderRoutes.js       # /api/stakeholders
│   ├── services/
│   │   ├── blockchainService.js       # Python blockchain + Fabric bridge
│   │   ├── fabricService.js           # Hyperledger Fabric mock
│   │   └── realFabricService.js       # Hyperledger Fabric SDK calls
│   ├── utils/
│   │   ├── encryption.js              # AES-256-GCM encrypt / decrypt
│   │   ├── refundCalculator.js        # Refund % by policy + hours remaining
│   │   └── captureLocation.js         # IP geolocation for login tracking
│   ├── scripts/
│   │   ├── seed-db.js                 # Create superadmin account
│   │   ├── seedStations.js            # Seed 50+ Bangladesh Railway stations
│   │   ├── seedRoutes.js              # Seed 8 routes
│   │   ├── seedTrains.js              # Seed 14 trains with schedules
│   │   ├── encryptExistingUsers.js    # Re-encrypt all sensitive user fields
│   │   └── create-indexes.js          # MongoDB performance indexes
│   ├── config/
│   │   └── database.js                # MongoDB connection setup
│   ├── data/
│   │   └── bangladeshRailwayData.js   # Station data source
│   ├── app.js                         # Express app + middleware setup
│   ├── server.js                      # HTTP server + Socket.IO init
│   └── Dockerfile
│
├── frontend/
│   ├── passenger-app/                 # Public booking portal  :3001
│   │   ├── src/App.js                 # Main app — search, book, QR, notify
│   │   ├── Dockerfile
│   │   ├── nginx.conf
│   │   └── package.json
│   │
│   ├── inspector-app/                 # On-board verification  :3002
│   │   ├── src/App.js                 # QR scan, verify, mark used
│   │   ├── Dockerfile
│   │   ├── nginx.conf
│   │   └── package.json
│   │
│   └── admin-dashboard/               # System management      :3003
│       ├── src/
│       │   ├── App.js                 # Main app — all admin features
│       │   └── pages/
│       │       ├── TrainManagement.js
│       │       ├── StationManagement.js
│       │       ├── RouteManagement.js
│       │       └── RefundPolicy.js
│       ├── Dockerfile
│       ├── nginx.conf
│       └── package.json
│
├── blockchain-python/                 # Custom PoW blockchain  :5001
│   ├── run.py                         # Flask app — all API endpoints
│   ├── adapters/
│   │   └── bitcoin_backend.py         # Bridge to core Blockchain class
│   ├── Blockchain/Backend/core/
│   │   ├── blockchain.py              # Block mining, wallet, TX management
│   │   ├── block.py                   # Block data structure
│   │   ├── blockheader.py             # PoW mining loop (nonce increment)
│   │   └── Tx.py                      # Coinbase transaction builder
│   ├── templates/                     # HTML block explorer templates
│   ├── data/
│   │   ├── blockchain.json            # The live blockchain
│   │   ├── users.json                 # Wallet users + bcrypt passwords
│   │   ├── wallets.json               # Wallet balances mirror
│   │   ├── mempool.json               # Pending transactions queue
│   │   └── tx_log.json                # Full transaction history
│   ├── requirements.txt
│   └── Dockerfile
│
├── blockchain/                        # Hyperledger Fabric (optional)
│   ├── chaincode/ticket_contract/     # Go smart contract
│   ├── config/                        # configtx, crypto-config, core YAML
│   ├── scripts/                       # start, stop, deploy shell scripts
│   ├── bin/                           # Fabric peer/orderer binaries
│   └── organizations/                 # MSP certificates and keys
│
├── docker-compose.yml                 # Full stack orchestration
├── .env                               # Environment variables (not in git)
├── .env.example                       # Environment template
└── README.md
```

---

*A Pathway to Cloud-Based Blockchain-Enabled Ticketing System for National Railway of Bangladesh — Node.js · React 18 · MongoDB · Python · Blockchain (PoW) · Hyperledger Fabric · Cloud*

---

## Copyright

&copy; 2026 **Rajib Das**  
M.Sc. in Cyber Security &bull; Bangladesh University of Professionals (BUP)  
[rajib.24525201034@student.bup.edu.bd](mailto:rajib.24525201034@student.bup.edu.bd)
