# Zynga Poker Chips Management Panel (Prototype)

This is a **multi-platform prototype panel** for managing Zynga Poker chips.  
It is a **mock/private system** designed for admin and player role management, real-time balances, transfers, and transaction logging.  
**Important:** This panel does **not connect to Zynga servers** and does **not generate real chips**. All data is simulated for testing and demonstration purposes.

---

## Features

### User Roles
- **Administrator**
  - Full access to all user balances
  - Transfer chips between users
  - Reverse transactions with audit reason
  - View transaction history and export CSV
- **Player**
  - View own balance
  - View transaction history
  - Initiate transfer requests

### Core Screens
1. **Balance** — shows current balance in real time
2. **Transfer** — create transfers, reverse transfers, or request transfer (player)
3. **History** — complete transaction log with timestamps, amount, from/to, admin ID, reason, and idempotency key

### Backend
- Node.js + Express API
- MongoDB database with **Decimal128** for large chip counts
- Real-time updates with **Socket.io**
- JWT authentication and role-based access control
- Idempotent transfer endpoints for safety
- Optional bulk transfers via CSV and background processing

### Frontend
- React.js (PWA-ready)
- Responsive UI (desktop + mobile)
- Admin and player dashboards

---

## Installation & Setup

### Prerequisites
- Node.js >= 18
- MongoDB >= 6
- Redis (optional, for caching & real-time)
- npm or yarn

### Backend Setup
1. Clone repo and navigate to backend folder:
```bash
cd backend
npm install
