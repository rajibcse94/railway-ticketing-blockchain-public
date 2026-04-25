# A Pathway to Cloud-Based Blockchain-Enabled Ticketing System for National Railway of Bangladesh
### Blockchain-Powered | Full-Stack | Real-Time | QR Tickets | Cloud-Ready

A production-grade railway ticketing platform built for Bangladesh Railway. Every ticket booking, cancellation, user registration, and login is permanently recorded on a real Proof-of-Work blockchain as a mined block. Passengers search trains, select seats, pay, and receive QR-coded digital tickets. Admins manage the entire railway operation in real-time. Inspectors verify tickets on board.

---

## Table of Contents

1. [System Overview](#1-system-overview)
2. [System Architecture — Layered View](#2-system-architecture--layered-view)
3. [Architecture — What the Blockchain Stores](#3-architecture--what-the-blockchain-stores)
4. [Architecture — Data Flow Diagram](#4-architecture--data-flow-diagram)
5. [Architecture — Component Diagram](#5-architecture--component-diagram)
6. [Architecture — Cloud Deployment Environment](#6-architecture--cloud-deployment-environment)
7. [Technology Stack](#7-technology-stack)
8. [Fresh Ubuntu 22.04 Setup — Full Guide](#8-fresh-ubuntu-2204-setup--full-guide)
9. [Environment Variables](#9-environment-variables)
10. [Start / Stop the System](#10-start--stop-the-system)
11. [Seed the Database](#11-seed-the-database)
12. [Default Logins & Ports](#12-default-logins--ports)
13. [User Roles & Permissions](#13-user-roles--permissions)
14. [How the System Works — End to End](#14-how-the-system-works--end-to-end)
15. [Ticket Booking Flowchart](#15-ticket-booking-flowchart)
16. [Refund & Cancellation Policy](#16-refund--cancellation-policy)
17. [Real-Time Updates — WebSocket](#17-real-time-updates--websocket)
18. [Notification System](#18-notification-system)
19. [Blockchain — How It Actually Works](#19-blockchain--how-it-actually-works)
20. [Security Features](#20-security-features)
21. [API Reference](#21-api-reference)
22. [Seed Scripts](#22-seed-scripts)
23. [Hyperledger Fabric (Optional)](#23-hyperledger-fabric-optional)
24. [Production Deployment](#24-production-deployment)
25. [Project Structure](#25-project-structure)

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
| **Cloud** | KVM VPS (Ubuntu 22.04) — Docker-orchestrated, Nginx reverse proxy, SSL |

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
║  • Cancel Ticket ║                      ║  • Blockchain Ledger (Live)        ║
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
║  • Socket.IO Server (real-time event broadcasting to admin dashboard)      ║
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
║ authController     ║ passengerCtrl    ║ adminController  ║ inspectorCtrl    ║
║                    ║                  ║                  ║                  ║
║ • register         ║ • searchTrains   ║ • manageTrains   ║ • verifyTicket   ║
║ • login            ║ • getSeats       ║ • manageRoutes   ║ • markUsed       ║
║ • MFA setup/verify ║ • bookTicket     ║ • getTickets     ║ • flagTicket     ║
║ • refreshToken     ║ • cancelTicket   ║ • cancelTrain    ║                  ║
║ • OTP send/verify  ║ • getMyTickets   ║ • notifyUsers    ║ superAdminCtrl   ║
║                    ║ • QR generate    ║ • revenueReport  ║                  ║
║ Emits:             ║ • waitlist       ║ • refundPolicy   ║ • createUser     ║
║  user:registered   ║ • notifications  ║                  ║ • editUser       ║
║  user:login        ║                  ║                  ║ • deleteUser     ║
║                    ║ Emits:           ║                  ║ • revenueView    ║
║                    ║  ticket:booked   ║                  ║                  ║
║                    ║  ticket:cancelled║                  ║                  ║
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
║  • recordUserLogin       ║  • Apply policy rules ║  • Store per-login       ║
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

## 3. Architecture — What the Blockchain Stores

Every event in the system mines a real SHA-256 PoW block. The block contains a transaction with the full event data — permanently immutable.

```
╔══════════════════════════════════════════════════════════════════════════════╗
║              WHAT GETS STORED IN THE BLOCKCHAIN                             ║
╠══════════════╦═══════════════════════════════════════════════════════════════╣
║  EVENT       ║  DATA STORED INSIDE THE BLOCK TRANSACTION                   ║
╠══════════════╬═══════════════════════════════════════════════════════════════╣
║              ║  type         : "USER_REGISTRATION"                         ║
║  NEW USER    ║  txid         : SHA-256(userId + email + timestamp)         ║
║  REGISTERS   ║  user_id      : MongoDB ObjectId of the new user            ║
║              ║  name         : Full name                                   ║
║              ║  email        : Email address                               ║
║              ║  role         : "passenger"                                 ║
║              ║  mfa          : MFA enabled true/false                      ║
║              ║  ip           : Registration IP address                     ║
║              ║  timestamp    : Unix timestamp                              ║
╠══════════════╬═══════════════════════════════════════════════════════════════╣
║              ║  type         : "USER_LOGIN"                                ║
║  USER        ║  txid         : SHA-256(userId + email + timestamp)         ║
║  LOGS IN     ║  user_id      : MongoDB ObjectId                            ║
║              ║  name         : Full name                                   ║
║              ║  email        : Email address                               ║
║              ║  role         : passenger / admin / inspector               ║
║              ║  ip           : Login IP address                            ║
║              ║  timestamp    : Unix timestamp                              ║
╠══════════════╬═══════════════════════════════════════════════════════════════╣
║              ║  type         : "TICKET_BOOKING"                            ║
║  TICKET      ║  txid         : SHA-256(ticket# + passengerId + timestamp)  ║
║  BOOKED      ║  ticket_number: TICKET-{timestamp}                          ║
║              ║  passenger_id : MongoDB ObjectId                            ║
║              ║  passenger_name: Full name                                  ║
║              ║  train_number : e.g. "711"                                  ║
║              ║  train_name   : e.g. "Subarna Express"                      ║
║              ║  from_station : Boarding station                            ║
║              ║  to_station   : Alighting station                           ║
║              ║  journey_date : YYYY-MM-DD                                  ║
║              ║  carriage_name: e.g. "Shovon-1"                             ║
║              ║  seat_numbers : [5, 6]                                      ║
║              ║  amount       : Fare paid (৳)                               ║
║              ║  data_hash    : SHA-256 integrity hash of ticket            ║
║              ║  timestamp    : Unix timestamp                              ║
╠══════════════╬═══════════════════════════════════════════════════════════════╣
║              ║  type            : "TICKET_CANCELLATION"                    ║
║  TICKET      ║  txid            : SHA-256(ticket# + passengerId + ts)      ║
║  CANCELLED   ║  ticket_number   : Original ticket number                   ║
║              ║  passenger_id    : MongoDB ObjectId                         ║
║              ║  passenger_name  : Full name                                ║
║              ║  train_number    : Train number                             ║
║              ║  journey_date    : YYYY-MM-DD                               ║
║              ║  from_station    : Boarding station                         ║
║              ║  to_station      : Alighting station                        ║
║              ║  reason          : Cancellation reason text                 ║
║              ║  original_amount : Original fare paid (৳)                   ║
║              ║  refund_amount   : Refund credited (৳)                       ║
║              ║  refund_percentage: e.g. 80.0                               ║
║              ║  timestamp       : Unix timestamp                           ║
╠══════════════╬═══════════════════════════════════════════════════════════════╣
║              ║  type     : "payment"                                       ║
║  WALLET      ║  txid     : Transaction ID                                  ║
║  PAYMENT     ║  sender   : User wallet ID                                  ║
║              ║  receiver : "railway_admin"                                 ║
║              ║  amount   : Amount deducted (৳)                             ║
║              ║  timestamp: Unix timestamp                                  ║
╚══════════════╩═══════════════════════════════════════════════════════════════╝

  EVERY BLOCK ALSO CONTAINS A COINBASE TRANSACTION:
  ┌──────────────────────────────────────────────┐
  │  { type: "coinbase", txid: "...", amount: 5000000000 }  │
  └──────────────────────────────────────────────┘

  BLOCK HEADER (each block):
  ┌──────────────────────────────────────────────────────────┐
  │  prevBlockHash  : hash of the previous block             │
  │  merkleRoot     : SHA-256 root of all transactions       │
  │  timestamp      : Unix time of mining                    │
  │  nonce          : incremented until hash starts with 7482│
  │  blockHash      : SHA-256(header) — starts with "7482"   │
  └──────────────────────────────────────────────────────────┘

  OFF-CHAIN vs ON-CHAIN SPLIT:
  ┌─────────────────────────────┬──────────────────────────────────────┐
  │  MongoDB (Off-chain)        │  Python Blockchain (On-chain)        │
  ├─────────────────────────────┼──────────────────────────────────────┤
  │  Full ticket document       │  Ticket booking event + hash         │
  │  Full user profile          │  User registration + login events    │
  │  Train/route/station data   │  Cancellation + refund record        │
  │  Notifications              │  Payment transactions                │
  │  OTPs, waitlists, logs      │  Immutable PoW-secured chain         │
  │  Mutable — can be queried   │  Append-only — tamper-proof          │
  └─────────────────────────────┴──────────────────────────────────────┘
```

---

## 4. Architecture — Data Flow Diagram

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
  │     (non-blocking, async)       │         → TICKET_BOOKING tx built
  │                                 │         → Added to mempool
  │                                 │         → Nonce++ until hash=7482...
  │                                 │         → Block appended to chain
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

  BLOCKCHAIN MINING FLOW
  ──────────────────────
  Any trigger (register / login / book / cancel)
       │
       ▼
  Backend POST → Python blockchain /api/record/*
       │
       ▼
  Build transaction dict
  { type, txid: SHA256(key_fields+ts), ...full event data }
       │
       ▼
  Add to mempool.json
       │
       ▼
  mine_next_block()
       │
       ▼
  Increment nonce until SHA256(header) starts with "7482"
       │
       ▼
  Append block to blockchain.json (permanent, immutable)
       │
       ▼
  Return { blockHeight, blockHash }
       │
       ▼
  Backend logs: "[BLOCKCHAIN] Ticket TICKET-xxx → block #N hash=7482..."
```

---

## 5. Architecture — Component Diagram

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
│  │  Volume: mongodb-data│              │  Volume: blockchain-data    │   │
│  └──────────────────────┘              └─────────────────────────────┘   │
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
  │  Block #N-1  blockHash: "7482a3b8f..."       │
  └──────────────────────────────────────────────┘
```

---

## 6. Architecture — Cloud Deployment Environment

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                    CLOUD ENVIRONMENT ARCHITECTURE                           ║
║           KVM VPS — Ubuntu 22.04 LTS — 8 Core · 12 GB RAM · 250 GB SSD    ║
╚══════════════════════════════════════════════════════════════════════════════╝

  INTERNET
      │
      │  HTTPS :443  /  HTTP :80
      ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                     NGINX REVERSE PROXY                                  │
│                  (SSL Termination · Load Balancing)                      │
│                                                                          │
│  yourdomain.com          → Passenger App  :3001                         │
│  yourdomain.com/inspect  → Inspector App  :3002                         │
│  admin.yourdomain.com    → Admin Dashboard :3003                        │
│  yourdomain.com/api/     → Backend API    :3000                         │
│  yourdomain.com/socket.io→ Socket.IO      :3000 (WebSocket upgrade)    │
│                                                                          │
│  SSL: Let's Encrypt (Certbot auto-renew)                                │
└──────────────────────────────────┬──────────────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                   DOCKER COMPOSE STACK                                   │
│                 (Internal Network: railway-network)                      │
│                                                                          │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────────────────┐   │
│  │ passenger-app │  │ inspector-app │  │    admin-dashboard        │   │
│  │ Nginx :3001   │  │ Nginx :3002   │  │    Nginx + Socket.IO      │   │
│  │ React 18      │  │ React 18      │  │    :3003                  │   │
│  └───────────────┘  └───────────────┘  └───────────────────────────┘   │
│                                                                          │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │               Node.js Backend  :3000                              │  │
│  │     Express · Socket.IO · JWT · Helmet · Joi · Mongoose           │  │
│  └──────────────────────┬──────────────────────┬──────────────────── ┘  │
│                         │                      │                         │
│  ┌──────────────────────┐    ┌──────────────────────────────────────┐   │
│  │  MongoDB  :27017     │    │  Python Blockchain  :5001            │   │
│  │  Volume: mongodb-data│    │  Volume: blockchain-data             │   │
│  │  (persistent)        │    │  (persistent — blockchain.json)      │   │
│  └──────────────────────┘    └──────────────────────────────────────┘   │
│                                                                          │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │  Hyperledger Fabric (Optional)                                    │  │
│  │  orderer0 :7050 · orderer1 :7052 · orderer2 :7053                │  │
│  │  peer0 :7051 · peer1 :8051 · couchdb0 · couchdb1                 │  │
│  └───────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘

  FIREWALL (UFW)
  ──────────────
  ✓ ALLOW   :22   SSH
  ✓ ALLOW   :80   HTTP  (Nginx)
  ✓ ALLOW   :443  HTTPS (Nginx + SSL)
  ✗ DENY    :3000  Block direct backend access
  ✗ DENY    :3001  Block direct app access
  ✗ DENY    :3002  Block direct app access
  ✗ DENY    :3003  Block direct app access
  ✗ DENY    :5001  Block direct blockchain access
  ✗ DENY    :27017 Block direct MongoDB access

  STORAGE VOLUMES (Docker persistent)
  ─────────────────────────────────────
  mongodb-data       → All MongoDB collections (tickets, users, trains...)
  blockchain-data    → blockchain.json, users.json, wallets.json, mempool.json

  BACKUP STRATEGY
  ────────────────
  MongoDB   → mongodump daily → compressed archive → off-site storage
  Blockchain→ copy blockchain-data volume → off-site storage
  .env      → encrypted backup → secure vault

  CLOUD SCALING PATH
  ───────────────────
  Current:  Single KVM VPS (8 core / 12 GB / 250 GB SSD)
  Scale-up: Increase VPS resources (vertical scaling)
  Scale-out: Add load balancer → multiple backend containers
             MongoDB Atlas (managed) → replace self-hosted MongoDB
             Separate blockchain node on dedicated VPS
             CDN for frontend static assets
```

---

## 7. Technology Stack

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

### Infrastructure & Cloud

| Tool | Purpose |
|---|---|
| Docker + Docker Compose | All services containerised and orchestrated |
| MongoDB 5 | Primary off-chain data store |
| Nginx | Reverse proxy + SSL termination + static serving |
| Let's Encrypt / Certbot | Free SSL certificates with auto-renewal |
| UFW | Ubuntu firewall — port whitelist |
| Hyperledger Fabric 2.x | Enterprise permissioned blockchain (optional) |
| CouchDB 3.1 | Hyperledger Fabric world-state database |
| Ubuntu 22.04 LTS | Cloud VPS operating system |
| KVM VPS | 8 core · 12 GB RAM · 250 GB SSD |

---

## 8. Fresh Ubuntu 22.04 Setup — Full Guide

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

Fill in all values — see [Section 9](#9-environment-variables).

### Step 7 — Build and start everything

```bash
docker compose up --build -d
```

### Step 8 — Seed the database

```bash
docker exec railway-backend node scripts/seed-db.js
docker exec railway-backend node scripts/seedStations.js
docker exec railway-backend node scripts/seedRoutes.js
docker exec railway-backend node scripts/seedTrains.js
```

### Step 9 — Verify

```bash
docker ps
curl http://localhost:3000/health      # OK
curl http://localhost:5001/health      # {"status":"ok"}
```

---

## 9. Environment Variables

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
MONGO_URI=mongodb://admin:<your@mongodb>:27017/railway-ticketing?authSource=admin
MONGO_ROOT_USERNAME=<mongodb username>
MONGO_ROOT_PASSWORD=<mongodb pass>

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
EMAIL_PASS=xxxx_xxxx_xxxx_xxxx

# ── CORS ─────────────────────────────────────────────────────
CORS_ORIGINS=http://localhost:3001,http://localhost:3002,http://localhost:3003

# ── CouchDB (Hyperledger Fabric) ─────────────────────────────
COUCHDB_USER=admin
COUCHDB_PASSWORD=adminpw
```

---

## 10. Start / Stop the System

```bash
docker compose up -d                        # Start all
docker compose down                         # Stop (data preserved)
docker compose down -v                      # Full reset (deletes data)
docker compose up --build -d               # Rebuild after code changes
docker compose up -d --build backend       # Rebuild one service
docker compose restart backend             # Restart one service
docker logs railway-backend -f             # Live logs
docker logs blockchain -f
```

---

## 11. Seed the Database

```bash
docker exec railway-backend node scripts/seed-db.js       # superadmin
docker exec railway-backend node scripts/seedStations.js  # 50+ stations
docker exec railway-backend node scripts/seedRoutes.js    # 8 routes
docker exec railway-backend node scripts/seedTrains.js    # 14 trains
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

## 12. Default Logins & Ports

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

## 13. User Roles & Permissions

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

## 14. How the System Works — End to End

```
REGISTER
  Enter: name, email, mobile, NID/passport, DOB, gender, password
  System sends 6-digit OTP to email (expires 10 minutes)
  Passenger enters OTP → account created
  Sensitive fields (NID, DOB, mobile) encrypted with AES-256-GCM
  Blockchain wallet created: ৳20,000 starting balance
  USER_REGISTRATION mined as PoW block → permanently on chain
  Optional: enable MFA (Google Authenticator)

LOGIN
  Enter email + password → MFA TOTP if enabled
  USER_LOGIN mined as PoW block → permanently on chain
  Receive JWT access token (15 min) + refresh token (7 days)

SEARCH TRAINS
  Enter: From → To → Date
  Queries train stops (intermediate stop support)
  Example: Dhaka→Comilla returns Subarna Express (Dhaka→Chittagong)

SELECT & BOOK
  Choose carriage class → seat layout (green=available, red=booked)
  Select 1–4 seats → confirm payment (direct or blockchain wallet)
  Ticket saved in MongoDB + TICKET_BOOKING mined as PoW block
  Admin dashboard updates INSTANTLY via Socket.IO

ON BOARD
  Inspector scans QR → verifies hash, train, date, status
  Marks as USED → MongoDB updated permanently

CANCEL
  Passenger sees refund estimate → confirms with password
  Refund credited to wallet → TICKET_CANCELLATION mined as PoW block
  Admin dashboard updates INSTANTLY via Socket.IO
```

---

## 15. Ticket Booking Flowchart

```
START
  │
  ▼
Enter: From Station → To Station → Date
  │
  ▼
Query trains (MongoDB $all on stops) + direction filter
  ├── No trains ───────────────────────► "No trains available"
  ▼
Select train → carriage → seat layout
  │
  ▼
Select 1–4 seats
  │
  ▼
Seat available?  ── NO ──────────────► "Seat already booked"
  │ YES
  ▼
Passenger < 4 active tickets?  ── NO ► "Max 4 seats limit"
  │ YES
  ▼
Blockchain wallet payment?
  ├── YES → Wallet password → deduct balance
  └── NO  → Proceed direct
  │
  ▼
Save Ticket → MongoDB
  │
  ▼
Generate SHA-256 integrity hash
  │
  ▼
Emit Socket.IO → Admin dashboard instant update
  │
  ▼
Mine TICKET_BOOKING block (async)
  nonce++ until SHA256(header) starts with "7482"
  │
  ▼
Return QR ticket to passenger
  │
END
```

---

## 16. Refund & Cancellation Policy

| Time Before Departure | Refund |
|---|---|
| More than 48 hours | 80 % |
| 24 to 48 hours | 50 % |
| 0 to 24 hours | 25 % |
| After departure | 0 % |

- Passenger sees exact refund amount before confirming
- Account password required to confirm
- Wallet payments → refund credited back automatically
- Every cancellation mined as a PoW block with full refund record
- Policy thresholds configurable by admin

---

## 17. Real-Time Updates — WebSocket

| Socket Event | Triggered When | Admin Dashboard Effect |
|---|---|---|
| `ticket:booked` | Passenger books ticket | Ticket counts, revenue, available seats update instantly |
| `ticket:cancelled` | Passenger cancels ticket | Cancelled count, seat availability restored instantly |
| `user:registered` | New passenger registers | Total users, passenger list refreshed |
| `user:login` | Any user logs in | Blockchain ledger stats refreshed |

Polling every 30 seconds runs as fallback for any missed events.

---

## 18. Notification System

```
Admin cancels/reschedules train
  → Backend finds all affected tickets + users
  → Creates Notification in MongoDB per user
  → Sends HTML email (red=cancelled, blue=rescheduled) via Gmail SMTP
  → Passenger app polls /notifications every 15 seconds
  → Bell badge shows unread count
  → Toast notification bottom-right
  → Click bell → panel opens → marked as read

Admin Notifications tab shows:
  User Name | Email | Role | Type | Train | Message | Email Sent | Read | Time
```

---

## 19. Blockchain — How It Actually Works

### Python Custom Proof-of-Work Blockchain (Port 5001)

This is not a simulation. It is a real blockchain with real SHA-256 Proof-of-Work mining. Every block hash starts with `7482`.

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
    { "type": "coinbase", "txid": "85c3a159...", "amount": 5000000000 },
    {
      "type": "TICKET_BOOKING",
      "txid": "e247a04f...",
      "ticket_number": "TICKET-1776843330",
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

**Block hash starts with `7482` — real Proof-of-Work.**

**Blockchain API Endpoints:**

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/record/ticket` | Mine ticket booking as block |
| POST | `/api/record/cancellation` | Mine cancellation as block |
| POST | `/api/record/user-register` | Mine user registration as block |
| POST | `/api/record/user-login` | Mine user login as block |
| GET | `/api/blocks` | Full chain as JSON |
| GET | `/api/chain/stats` | Total blocks, bookings, cancellations, logins |
| POST | `/api/blockchain/pay` | Deduct fare from wallet |
| POST | `/api/wallet` | Create wallet on registration |
| GET | `/api/wallet/<userId>` | Wallet balance + info |
| POST | `/api/wallet/refund` | Credit refund to wallet |
| GET | `/api/wallets` | All wallet balances |
| GET | `/blocks` | Block explorer web UI |

**Wallet System:**
- Every passenger gets a blockchain wallet on registration (৳ 20,000 starting balance)
- Wallet password stored bcrypt-hashed in `users.json`
- Refunds credited back automatically on cancellation

### Hyperledger Fabric (Optional)

| Component | Role |
|---|---|
| Certificate Authority | Issues MSP identities (port 7054) |
| 3 Orderers (Raft) | Transaction ordering |
| 2 Peers | Ledger + chaincode execution |
| 2 CouchDB | World-state databases |
| Channel | `railwaychannel` |
| Chaincode | `ticket` (Go) |

Enable: `USE_REAL_FABRIC=true` in `.env` → restart backend.

---

## 20. Security Features

| Category | Feature | Implementation |
|---|---|---|
| Auth | Access token | JWT 15-min, signed HS256 |
| Auth | Refresh token | JWT 7-day |
| Auth | Auto-refresh | Axios interceptor — silent retry on 401 |
| Auth | MFA | TOTP via speakeasy (Google Authenticator) |
| Encryption | Passwords | bcrypt cost 10 |
| Encryption | NID / Passport | AES-256-GCM, random IV per value |
| Encryption | Date of birth | AES-256-GCM |
| Encryption | Mobile number | AES-256-GCM |
| Integrity | Ticket hash | SHA-256 stored in MongoDB + blockchain |
| Network | Headers | Helmet: XSS, X-Frame, HSTS, CSP |
| Network | CORS | Whitelist only via CORS_ORIGINS |
| Network | Validation | Joi — unknown fields stripped |
| Audit | Login tracking | IP → city/country, stored per login |
| Audit | Login history | Last 20 logins per user |
| Audit | Profile changes | Before/after logged in profileupdatelogs |
| Blockchain | Immutability | All events as PoW blocks — tamper-proof |

---

## 21. API Reference

### Auth `/api/auth`

| Method | Path | Auth | Description |
|---|---|---|---|
| POST | `/register/start` | None | Send OTP to email |
| POST | `/register/verify` | None | Verify OTP, activate account |
| POST | `/login` | None | Login — access + refresh tokens |
| POST | `/refresh` | None | Refresh access token |
| GET | `/me` | JWT | Current user profile |

### Passenger `/api/passenger`

| Method | Path | Description |
|---|---|---|
| GET | `/trains/search?from=&to=&date=` | Search trains (intermediate stops) |
| GET | `/seats?trainId=&date=&carriageName=` | Real-time seat availability |
| POST | `/tickets/book` | Book ticket |
| GET | `/tickets/my` | My tickets |
| POST | `/tickets/:id/cancel` | Cancel (password required) |
| GET | `/tickets/:id/refund-estimate` | Refund preview |
| GET | `/tickets/:id/qr` | QR code |
| GET | `/tickets/:id/verify` | Verify integrity hash |
| GET | `/notifications` | My notifications |
| PUT | `/notifications/read` | Mark all read |
| PUT | `/profile/update` | Update profile (OTP required) |
| GET | `/wallet/transactions` | Wallet history |

### Admin `/api/admin`

| Method | Path | Description |
|---|---|---|
| GET | `/tickets` | All tickets |
| GET | `/passengers` | All passengers + login history |
| GET | `/trains` | All trains |
| POST | `/trains/:id/cancel-reschedule` | Cancel/reschedule + notify |
| GET | `/notifications/all` | All notifications |
| GET | `/revenue` | Revenue reports |
| GET | `/blockchain/chain` | Live blockchain |
| GET | `/blockchain/stats` | Chain statistics |
| GET | `/blockchain/wallets` | All wallet balances |

### Super Admin `/api/superadmin`

| Method | Path | Description |
|---|---|---|
| GET | `/users` | All users + decrypted fields |
| POST | `/users` | Create user (any role) |
| PUT | `/users/:id` | Edit / reset MFA / change role |
| DELETE | `/users/:id` | Delete user |

### Inspector `/api/inspector`

| Method | Path | Description |
|---|---|---|
| POST | `/tickets/verify` | Verify QR ticket |
| POST | `/tickets/:id/use` | Mark as USED |
| POST | `/tickets/:id/flag` | Flag suspicious ticket |

---

## 22. Seed Scripts

```bash
docker exec railway-backend node scripts/seed-db.js
# Creates: superadmin@railway.com / Admin@123

docker exec railway-backend node scripts/seedStations.js
# 50+ stations with GPS, division, zone

docker exec railway-backend node scripts/seedRoutes.js
# 8 routes (Dhaka–Chittagong, Dhaka–Sylhet, Dhaka–Khulna,
#           Dhaka–Rajshahi, Dhaka–Mymensingh, Ctg–Cox's Bazar,
#           Dhaka–Rajshahi via Santahar, Rajshahi–Rohanpur)

docker exec railway-backend node scripts/seedTrains.js
# 14 trains (7 routes × up/down) — all carriages + 30-day schedules

docker exec railway-backend node scripts/encryptExistingUsers.js
# Re-encrypt sensitive fields (after ENCRYPTION_KEY change)

docker exec railway-backend node scripts/create-indexes.js
# MongoDB performance indexes
```

---

## 23. Hyperledger Fabric (Optional)

```bash
cd blockchain
bash scripts/generate-certs.sh
bash scripts/start-network.sh
bash scripts/create-channel.sh
bash scripts/deploy-chaincode.sh
cd ..
node scripts/enroll-users.js
# Set USE_REAL_FABRIC=true in .env
docker compose restart backend
```

---

## 24. Production Deployment

### VPS Requirements

| Resource | Minimum | Recommended |
|---|---|---|
| CPU | 4 cores | 8 cores |
| RAM | 8 GB | 12 GB |
| Disk | 50 GB SSD | 250 GB SSD |
| OS | Ubuntu 22.04 LTS | Ubuntu 22.04 LTS |

### Generate Secure Keys

```bash
node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"
# Run 3 times: JWT_SECRET, JWT_REFRESH_SECRET, ENCRYPTION_KEY
```

### Nginx Reverse Proxy

```nginx
server {
    listen 80;
    server_name yourdomain.com;

    location /api/ {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /socket.io/ {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }

    location / { proxy_pass http://localhost:3001; }
}
```

### SSL, Firewall, Boot

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d yourdomain.com

sudo ufw allow 22/tcp && sudo ufw allow 80/tcp && sudo ufw allow 443/tcp
sudo ufw deny 3000/tcp && sudo ufw deny 5001/tcp && sudo ufw deny 27017/tcp
sudo ufw enable

sudo systemctl enable docker
```

### MongoDB Backup

```bash
docker exec mongodb mongodump \
  --uri="mongodb://admin:PASSWORD@localhost:27017/railway-ticketing?authSource=admin" \
  --out=/backup/$(date +%Y%m%d)
```

---

## 25. Project Structure

```
railway-ticketing-blockchain/
│
├── backend/
│   ├── controllers/
│   │   ├── authController.js          # register, login, MFA, OTP
│   │   ├── passengerController.js     # search, book, cancel, QR
│   │   ├── adminController.js         # train mgmt, notify users
│   │   ├── inspectorController.js     # verify, mark used, flag
│   │   └── superAdminController.js    # user CRUD, revenue
│   ├── middleware/
│   │   ├── auth.js                    # JWT verify
│   │   ├── authorize.js               # RBAC role check
│   │   ├── validate.js                # Joi schemas
│   │   └── errorHandler.js            # Global error handler
│   ├── models/
│   │   ├── User.js                    # AES-256 fields, login history
│   │   ├── Ticket.js                  # SHA-256 hash, refund fields
│   │   ├── Train.js                   # carriages, schedules, stops
│   │   ├── Route.js                   # ordered stops
│   │   ├── Station.js                 # GPS, division, zone
│   │   ├── Notification.js            # emailSent, isRead
│   │   ├── OTP.js                     # expires 10 min
│   │   ├── WaitList.js                # per train/date/carriage
│   │   ├── RefundPolicy.js            # configurable thresholds
│   │   ├── Stakeholder.js             # API stakeholders
│   │   └── ProfileUpdateLog.js        # audit trail
│   ├── routes/
│   │   ├── authRoutes.js
│   │   ├── passengerRoutes.js
│   │   ├── inspectorRoutes.js
│   │   ├── adminRoutes.js             # + blockchain proxy routes
│   │   ├── superAdminRoutes.js
│   │   └── stakeholderRoutes.js
│   ├── services/
│   │   ├── blockchainService.js       # Python blockchain bridge
│   │   ├── fabricService.js           # Fabric mock
│   │   └── realFabricService.js       # Fabric SDK
│   ├── utils/
│   │   ├── encryption.js              # AES-256-GCM
│   │   ├── refundCalculator.js        # refund % by policy
│   │   └── captureLocation.js         # IP geolocation
│   ├── scripts/
│   │   ├── seed-db.js
│   │   ├── seedStations.js
│   │   ├── seedRoutes.js
│   │   ├── seedTrains.js
│   │   ├── encryptExistingUsers.js
│   │   └── create-indexes.js
│   ├── app.js                         # Express + middleware
│   ├── server.js                      # HTTP + Socket.IO
│   └── Dockerfile
│
├── frontend/
│   ├── passenger-app/    :3001        # Search, book, QR, notify
│   ├── inspector-app/    :3002        # Scan, verify, mark used
│   └── admin-dashboard/  :3003        # Full management + ledger
│       └── src/pages/
│           ├── TrainManagement.js
│           ├── StationManagement.js
│           ├── RouteManagement.js
│           └── RefundPolicy.js
│
├── blockchain-python/    :5001
│   ├── run.py                         # Flask API + all endpoints
│   ├── adapters/bitcoin_backend.py    # Bridge to Blockchain class
│   ├── Blockchain/Backend/core/
│   │   ├── blockchain.py              # Mining, wallet, TX
│   │   ├── blockheader.py             # PoW loop (hash starts 7482)
│   │   └── Tx.py                      # Coinbase builder
│   └── data/
│       ├── blockchain.json            # Live blockchain
│       ├── users.json                 # Wallets + bcrypt passwords
│       ├── wallets.json               # Balances
│       └── mempool.json               # Pending transactions
│
├── blockchain/                        # Hyperledger Fabric (optional)
│   ├── chaincode/ticket_contract/     # Go smart contract
│   ├── config/                        # configtx, crypto-config
│   └── scripts/                       # start, stop, deploy
│
├── docker-compose.yml
├── .env
├── .env.example
└── README.md
```

---

*A Pathway to Cloud-Based Blockchain-Enabled Ticketing System for National Railway of Bangladesh — Node.js · React 18 · MongoDB · Python · Blockchain (PoW) · Hyperledger Fabric · Cloud*

---

## Copyright

&copy; 2026 **Rajib Das**  
M.Sc. in Cyber Security &bull; Bangladesh University of Professionals (BUP)  
[rajib.24525201034@student.bup.edu.bd](mailto:rajib.24525201034@student.bup.edu.bd)

## License
This project is licensed under the MIT License - see the LICENSE file for details.
