# IDURAR ERP/CRM - Complete Codebase Overview

**Executive Summary: As Explained by CTO to Development Team**

---

## 📋 Table of Contents

1. [What is IDURAR?](#what-is-idurar)
2. [Technology Stack](#technology-stack)
3. [Architecture Overview](#architecture-overview)
4. [Core Features](#core-features)
5. [Project Structure](#project-structure)
6. [Development Workflow](#development-workflow)

---

## What is IDURAR?

IDURAR is an **Open Source ERP/CRM application** built for small to medium businesses to manage their:
- **Customers** (CRM functionality)
- **Invoices** (Billing and accounting)
- **Quotes** (Sales proposals)
- **Payments** (Payment tracking)
- **Financial reporting** (Dashboard and analytics)

### Key Characteristics:
- **Full-stack JavaScript** application (MERN stack)
- **Monorepo** structure with separate backend and frontend
- **RESTful API** architecture
- **JWT-based** authentication
- **MongoDB** for data persistence
- **React + Redux** for UI and state management
- **Ant Design** for professional UI components

---

## Technology Stack

### Backend (Node.js/Express)
```
Technology          Version     Purpose
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Node.js             20.9.0      Runtime environment
Express.js          4.18.2      Web application framework
MongoDB             Latest      NoSQL database
Mongoose            8.1.1       MongoDB ODM
JWT                 9.0.2       Authentication tokens
Bcrypt.js           2.4.3       Password hashing
Joi                 17.11.0     Schema validation
Multer              1.4.4       File upload handling
html-pdf            3.0.1       PDF generation
Pug                 3.0.2       Template engine
Resend              2.0.0       Email service
AWS SDK S3          3.509.0     Cloud storage
```

### Frontend (React)
```
Technology          Version     Purpose
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
React               18.3.1      UI library
Vite                5.4.8       Build tool & dev server
Redux Toolkit       2.2.1       State management
React Router v6     6.22.0      Client-side routing
Ant Design          5.14.1      UI component library
Axios               1.6.2       HTTP client
Day.js              1.11.10     Date manipulation
React Quill         2.0.0       Rich text editor
Currency.js         2.0.4       Financial calculations
```

---

## Architecture Overview

```
┌────────────────────────────────────────────────────┐
│           React Frontend (SPA)                     │
│   - Ant Design UI                                  │
│   - Redux State Management                         │
│   - React Router                                   │
└──────────────────┬─────────────────────────────────┘
                   │ REST API (JWT Auth)
                   ▼
┌────────────────────────────────────────────────────┐
│           Express.js Backend                       │
│   - JWT Middleware                                 │
│   - Controllers (Business Logic)                   │
│   - Mongoose Models                                │
└──────────────────┬─────────────────────────────────┘
                   │ Mongoose ODM
                   ▼
┌────────────────────────────────────────────────────┐
│           MongoDB Database                         │
│   - Collections: invoices, clients, quotes, etc.   │
└────────────────────────────────────────────────────┘
```

---

## Core Features

### 1. 🏢 Customer Management (CRM)
- Add, edit, delete customers
- Contact information storage
- Customer assignment to sales reps
- Search and filtering

### 2. 📄 Invoice Management
- Create professional invoices
- Line items with calculations
- Tax and discount support
- PDF generation
- Email invoices
- Payment tracking
- Status management (draft, sent, paid, etc.)

### 3. 💬 Quote Management
- Create sales quotes
- Convert quotes to invoices
- Email quotes to clients
- Similar to invoice workflow

### 4. 💰 Payment Management
- Record payments against invoices
- Multiple payment methods
- Partial payment support
- Payment history

### 5. 💳 Payment Modes
- Configure payment methods
- Cash, credit card, bank transfer, etc.

### 6. 📊 Tax Management
- Configure tax rates
- Apply to invoices/quotes

### 7. ⚙️ Settings Management
- Company information
- Invoice configuration
- Email settings
- Branding (logo)

### 8. 👥 User Management
- Admin accounts
- JWT authentication
- Profile management
- Password reset

### 9. 📁 File Management
- File uploads
- AWS S3 or local storage

### 10. 📧 Email System
- Transactional emails
- Invoice/quote delivery
- Password reset

### 11. 📊 Dashboard
- Revenue metrics
- Invoice statistics
- Payment tracking

---

## Project Structure

### Backend (`/backend/src/`)
```
backend/src/
├── server.js              # Entry point
├── app.js                 # Express config
├── models/                # Mongoose schemas
│   ├── appModels/        # Business models
│   └── coreModels/       # System models
├── controllers/           # Business logic
│   ├── appControllers/   # Entity controllers
│   └── coreControllers/  # System controllers
├── routes/                # API routes
├── middlewares/           # Express middlewares
├── pdf/                   # PDF templates
└── emailTemplate/         # Email templates
```

### Frontend (`/frontend/src/`)
```
frontend/src/
├── main.jsx               # Entry point
├── pages/                 # Page components
├── modules/               # Feature modules
├── redux/                 # State management
├── router/                # Routing
├── components/            # Shared components
└── request/               # API client
```

---

## Development Workflow

### Setup

**Backend:**
```bash
cd backend
npm install
npm run setup      # Initialize database
npm run dev        # Start dev server
```

**Frontend:**
```bash
cd frontend
npm install
npm run dev        # Start Vite dev server
```

### Key Endpoints
- Frontend: `http://localhost:3000`
- Backend API: `http://localhost:8888/api`

---

## Key Patterns

1. **RESTful API**: Consistent `/api/{entity}/{action}` structure
2. **JWT Auth**: Token-based authentication
3. **Soft Delete**: Records marked as `removed: true`
4. **Model-Controller**: Clear separation of data and logic
5. **Redux Toolkit**: Centralized state with async thunks
6. **Module Pattern**: Features as self-contained modules

---

## Next Documentation

- **[FEATURES_DETAILED.md](./FEATURES_DETAILED.md)** - Deep dive into each feature
- **[BACKEND_GUIDE.md](./BACKEND_GUIDE.md)** - Backend architecture details
- **[FRONTEND_GUIDE.md](./FRONTEND_GUIDE.md)** - Frontend architecture details
- **[NEXTJS_MIGRATION.md](./NEXTJS_MIGRATION.md)** - Migrate to Next.js guide

---

**This is a production-ready, well-architected application. Understanding these fundamentals will help you work effectively with the codebase.**
