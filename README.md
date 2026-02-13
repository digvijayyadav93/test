# AI Chatbot Appointment Booking System

A simplified end-to-end AI chatbot system for appointment booking, built with React/Next.js frontend, Node.js/Express backend, Python/LangChain AI microservice, and PostgreSQL database.

## 🎥 Demo Video

[![Watch the Demo](https://img.youtube.com/vi/RIpBCX-dYEs/0.jpg)](https://www.youtube.com/watch?v=RIpBCX-dYEs)

> **[Watch the full walkthrough on YouTube](https://youtu.be/RIpBCX-dYEs)**

## 📋 Table of Contents

- [High-Level Architecture](#high-level-architecture)
- [Technology Stack](#technology-stack)
- [Local Setup](#local-setup)
- [Key Design Decisions](#key-design-decisions)
- [Assumptions and Limitations](#assumptions-and-limitations)
- [API Documentation](#api-documentation)

---

## 🏗️ High-Level Architecture

```
┌─────────────┐
│   Browser   │
└──────┬──────┘
       │ HTTPS
       ▼
┌─────────────────────────┐
│   Next.js Frontend      │
│   - Authentication UI   │
│   - Chat Interface      │
│   - State Management    │
└───────────┬─────────────┘
            │ REST API + JWT
            ▼
┌─────────────────────────┐
│  Node.js/Express API    │
│  - JWT Authentication   │
│  - Session Management   │
│  - Appointment CRUD     │
│  - Rate Limiting        │
└───┬──────────────┬──────┘
    │              │
    │ HTTP         │ SQL
    ▼              ▼
┌───────────┐  ┌──────────┐
│  Python   │  │PostgreSQL│
│  LangChain│  │ Database │
│  AI Agent │  └──────────┘
└─────┬─────┘
      │ OpenAI API
      ▼
┌───────────┐
│  OpenAI   │
│    LLM    │
└───────────┘
```

## 🛠️ Technology Stack

### Frontend
- **Framework**: Next.js 14 (React 18)
- **Styling**: Tailwind CSS
- **HTTP Client**: Axios
- **State Management**: React Context API

### Backend API
- **Runtime**: Node.js
- **Framework**: Express
- **Authentication**: JWT (jsonwebtoken)
- **Database**: PostgreSQL with pg driver
- **Validation**: express-validator
- **Security**: bcrypt, rate limiting, CORS

### AI Microservice
- **Language**: Python 3.9+
- **Framework**: FastAPI
- **AI Orchestration**: LangChain
- **LLM Provider**: OpenAI GPT-3.5-turbo
- **Server**: Uvicorn

### Database
- **Database**: PostgreSQL 14+
- **Features**: UUID keys, JSONB for metadata, GIN indexes

---

## 🚀 Local Setup

### Prerequisites

Ensure you have the following installed:
- **Node.js** 18+ ([Download](https://nodejs.org/))
- **Python** 3.9+ ([Download](https://python.org/))
- **PostgreSQL** 14+ ([Download](https://www.postgresql.org/download/))
- **npm** (comes with Node.js)

### 1. Database Setup

1. Start PostgreSQL service
2. Create database:
```bash
createdb ai_chatbot_db
```

3. Run schema:
```bash
psql -d ai_chatbot_db -f database/schema.sql
```

4. (Optional) Load sample data:
```bash
psql -d ai_chatbot_db -f database/seed.sql
```

### 2. Backend API Setup

```bash
# Navigate to backend directory
cd backend

# Install dependencies
npm install

# Create .env file
cp .env.example .env

# Edit .env and add your database credentials
# DB_PASSWORD=your_postgres_password
# JWT_SECRET=your_secret_key_here

# Start server
npm run dev
```

Server will run on **http://localhost:3001**

### 3. AI Microservice Setup

```bash
# Navigate to AI service directory
cd ai-service

# Create virtual environment (recommended)
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Create .env file
cp .env.example .env

# Edit .env and add your OpenAI API key
# OPENAI_API_KEY=sk-your-api-key-here

# Start server
python main.py
```

Server will run on **http://localhost:8000**

### 4. Frontend Setup

```bash
# Navigate to frontend directory
cd frontend

# Install dependencies
npm install

# Create .env.local file
echo "NEXT_PUBLIC_API_URL=http://localhost:3001" > .env.local

# Start development server
npm run dev
```

Frontend will run on **http://localhost:3000**

### 5. Access the Application

1. Open browser to **http://localhost:3000**
2. Sign up or login (demo: john.doe@example.com / password123)
3. Start chatting to book appointments!

---

## 💡 Key Design Decisions

### 1. Microservices Architecture

**Decision**: Separate Python AI service from Node.js backend

**Rationale**:
- Python has superior AI/ML ecosystem (LangChain, extensive LLM integrations)
- Node.js excels at API serving and has broader deployment options
- Clean separation of concerns allows independent scaling

**Tradeoff**: Slightly more complex deployment, but better maintainability and scalability

### 2. JWT Authentication

**Decision**: Stateless JWT tokens instead of session-based auth

**Rationale**:
- No server-side session storage required
- Easier horizontal scaling
- Works well with microservices architecture
- Mobile-friendly

**Tradeoff**: Cannot revoke tokens before expiry (mitigation: short expiry times + refresh tokens in production)

### 3. HTTP Polling vs WebSockets

**Decision**: HTTP requests for chat instead of WebSockets

**Rationale**:
- Simpler implementation and debugging
- Easier deployment (no persistent connections)
- Sufficient for chatbot use case (not real-time gaming)
- Can upgrade to WebSockets later if needed

**Tradeoff**: Slightly higher latency (~200-500ms), but acceptable for conversational AI

### 4. LangChain Agent Pattern

**Decision**: Use LangChain's function-calling agents with custom tools

**Rationale**:
- Natural language understanding built-in
- Easy tool integration for appointment operations
- Conversation memory handling out of the box
- Flexible prompt engineering

**Tradeoff**: Dependent on OpenAI's function-calling feature (can use other providers)

9. **Limited Admin Panel**: No UI for business to manage availability/settings (Database access required)
10. **No Appointment Reminders**: No automated reminders before appointments

### 5. Real-Time Availability Checking
**Decision**: Check availability against global database state

**Rationale**:
- Prevents double-booking across concurrent users
- Enforces business hours and weekend closures logic
- Scalable to multiple frontend instances

**Tradeoff**: Requires database round-trip (negligible latency with indexing)

### 6. PostgreSQL with JSONB
**Decision**: Use PostgreSQL with JSONB for flexible metadata

**Rationale**:
- Relational data (users, appointments) fits well in SQL
- JSONB allows flexible conversation context storage
- GIN indexes enable fast JSONB queries
- Battle-tested, reliable database

**Tradeoff**: More setup than MongoDB, but better for structured data

---

## 📝 Assumptions
1. **Single Business Context**: System serves one business/organization (no multi-tenancy)
2. **Business Hours**: Fixed 9 AM - 5 PM, Monday-Friday
3. **Appointment Duration**: Default 30 minutes
4. **Time Zone**: Server's local time zone (no multi-timezone support)
5. **Concurrent Bookings**: Verified via database constraints
6. **LLM Provider**: OpenAI API available (user provides key)
7. **Local Development**: PostgreSQL installed and running locally
8. **No Payment**: Free appointment booking (no Stripe/payment integration)
9. **English Only**: Chatbot operates in English
10. **Desktop/Laptop**: Optimized for desktop browsers (responsive design would be added for production)

---

## ⚠️ Known Limitations
1. **No Calendar Integration**: Doesn't sync with Google Calendar, Outlook, etc.
2. **No Email Notifications**: Booking confirmations not sent via email
3. **No Rescheduling UI**: Reschedule must be done through chat (Implemented ✅)
4. **No Multi-tenancy**: Single business only
5. **Limited NLP**: Relies on LLM; may misinterpret complex requests
6. **No Timezone Support**: All times in server timezone
7. **Rate Limiting**: Per-User (JWT) and Per-IP protection enabled
8. **No Admin Panel**: No UI for business to manage availability/settings
9. **Chat History**: Persisted indefinitely (no archiving strategy yet)
10. **No Appointment Reminders**: No automated reminders before appointments

---

## 📡 API Documentation

### Authentication Endpoints

#### POST /api/auth/signup
Register a new user

**Request**:
```json
{
  "email": "user@example.com",
  "password": "password123",
  "name": "John Doe",
  "phone": "+1-555-0123"
}
```

**Response**:
```json
{
  "message": "User created successfully",
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "name": "John Doe"
  },
  "token": "jwt-token"
}
```

#### POST /api/auth/login
Authenticate user

**Request**:
```json
{
  "email": "user@example.com",
  "password": "password123"
}
```

### Chatbot Endpoints

#### POST /api/chatbot/token
Generate short-lived AI access token

**Headers**: `Authorization: Bearer <jwt-token>`

**Response**:
```json
{
  "token": "short-lived-token",
  "sessionId": "uuid",
  "expiresIn": 300
}
```

#### POST /api/chatbot/message
Send message to chatbot

**Headers**: `Authorization: Bearer <jwt-token>`

**Request**:
```json
{
  "message": "I'd like to book an appointment for tomorrow at 2 PM",
  "sessionId": "uuid"
}
```

**Response**:
```json
{
  "message": "I've booked your appointment for tomorrow at 2:00 PM",
  "metadata": {
    "intent": "book_appointment"
  },
  "sessionId": "uuid"
}
```

### Appointments Endpoints

#### GET /api/appointments
Get user's appointments

**Headers**: `Authorization: Bearer <jwt-token>`

**Query Parameters**:
- `status`: Filter by status (scheduled, completed, cancelled)
- `from_date`: Filter by start date (YYYY-MM-DD)
- `to_date`: Filter by end date (YYYY-MM-DD)

#### POST /api/appointments
Create appointment

**Headers**: `Authorization: Bearer <jwt-token>`

**Request**:
```json
{
  "appointment_date": "2024-03-15",
  "appointment_time": "14:00",
  "duration_minutes": 30,
  "notes": "Initial consultation"
}
```

---

## 🔄 Next Steps for Production

1. **Add WebSocket support** for real-time chat
2. **Integrate with Google Calendar** / Outlook for real availability
3. **Implement email notifications** (Resend, SendGrid)
4. **Add multi-tenancy** support for SaaS model
5. **Timezone support** for international users
6. **Admin dashboard** for business settings
7. **Appointment reminders** via email/SMS
8. **Payment integration** (Stripe) for paid appointments
9. **Analytics dashboard** for conversation insights
10. **Mobile app** (React Native) for iOS/Android

---

## 📄 License

MIT License - feel free to use for your projects!

---

## 🤝 Support

For questions or issues, please refer to the architecture documentation in `ARCHITECTURE.md`.
