# interview-tickets-ticketing-server

> An Express REST API service providing role-based issue ticket tracking, nested message communications, Mongoose ODM persistence, and Joi payload validation.

**Inception (First Commit):** `2022-07-08` • **Batch / Horizon:** `batch-2021-2022` • **Architecture Domain:** `Relational REST Backend / JWT & Sequelize`

---

## 📌 Architecture & Overview

`JobInterview_tickets-server` was engineered as a full-featured ticketing backend assessment service. Despite the initial repository metadata indexing it under relational database paradigms, the implementation is built upon MongoDB and Mongoose ODM. It exposes role-based ticket management, allowing regular users to create tickets and post communication messages while restricting status transitions and ticket deletions to administrator roles.

The server operates alongside its companion frontend SPA, [`interview-tickets-ticketing-client`](https://github.com/fabogit/interview-tickets-ticketing-client).

```
                                  +-----------------------------------------------+
                                  |         JobInterview_tickets-server           |
                                  |                 (Port: 3000)                  |
                                  |                                               |
[ Angular Client ]                |  +--------------------+                       |
(JobInterview_tickets-client)     |  | Helmet & Morgan    |                       |
        |                         |  +---------+----------+                       |
        | HTTP (Bearer JWT)       |            |                                  |
        v                         |  +---------v----------+                       |
+-------------------------------+ |  | CORS & JSON Parser |                       |
| Express (Port: 3000)          +-+->+---------+----------+                       |
+-------------------------------+ |            |                                  |
                                  |            v                                  |
                                  |  +--------------------+                       |
                                  |  | jwtMiddleware      |                       |
                                  |  | (AuthenticateUser) |                       |
                                  |  +---------+----------+                       |
                                  |            |                                  |
                                  |            v                                  |
                                  |  +--------------------+                       |
                                  |  | Joi Validators     |                       |
                                  |  | (Body / Params)    |                       |
                                  |  +---------+----------+                       |
                                  |            |                                  |
                                  |            v                                  |
                                  |  +--------------------+  +------------------+ |
                                  |  | Ticket Controllers |->| Mongoose ODM     | |
                                  |  | & Message Pushes   |  | (MongoDB)        | |
                                  |  +--------------------+  +------------------+ |
                                  +-----------------------------------------------+
```

* **Context:** Technical interview assignment evaluating full-stack authentication, role-based authorization, hierarchical document relationships (tickets with embedded communications), and defensive input validation.
* **Key Patterns:**
  * **Role-Based Access Control (RBAC):** Distinct permission hierarchies embedded within signed JWT payloads (`isAdmin: true/false`), restricting mutation endpoints (`PUT`, `DELETE`).
  * **Embedded Document Messaging:** Storing audit communications and user replies directly within an embedded array (`comunications`) on the parent `Ticket` document.
  * **Centralized Schema Validation:** Request payload and parameter verification enforced through declarative Joi schemas (`joi.object().keys(...)`).
  * **Paired Client-Server Integration:** Designed specifically to interface with [`interview-tickets-ticketing-client`](https://github.com/fabogit/interview-tickets-ticketing-client).

---

## 🛠 Tech Stack

* **Runtime & Framework:** Node.js (>= 16), Express ~4.16.1
* **Database & ODM:** MongoDB, Mongoose ^6.4.3
* **Authentication & Security:** `jsonwebtoken` ^8.5.1, `bcrypt` ^5.0.1, `helmet` ^5.1.0, `cors` ^2.8.5
* **Validation & Logging:** `joi` ^17.6.0, `morgan` ~1.9.1, `dotenv` ^16.0.1
* **Development Tooling:** `nodemon` ^2.0.19

---

## ✨ Features & Endpoints

* **User Authentication:** Registration and credential validation issuing signed HMAC-SHA256 JWT tokens.
* **Paginated Ticket Ledger:** Query filtering, pagination offsets, and total document count resolution.
* **Threaded Ticket Communications:** Appending chronological message entries to tickets via atomic `$push` updates.
* **Role Enforcement:** Administrative barriers protecting ticket status toggles (`isCompleted`) and deletions.

### Key API Routes / Interfaces

| Method | Route | Auth / Role | Description |
|---|---|---|---|
| `GET` | `/` | Public | Connectivity probe verifying service uptime. |
| `POST` | `/users` | Public | Register a new user account with hashed password. |
| `POST` | `/users/login` | Public | Authenticate user credentials and return signed JWT. |
| `GET` | `/tickets` | User / Admin | Retrieve paginated list of tickets (filtered by user if not admin). |
| `GET` | `/tickets/:ticketId` | User / Admin | Retrieve specific ticket by its MongoDB `_id`. |
| `POST` | `/tickets` | User / Admin | Create a new ticket entry. |
| `POST` | `/tickets/messages/:ticketId` | Owner / Admin | Append a conversation message to a ticket thread. |
| `PUT` | `/tickets/:ticketId` | Admin Only | Toggle ticket completion status (`isCompleted`). |
| `DELETE` | `/tickets/:ticketId` | Admin Only | Permanently delete a ticket document. |

---

## 🚀 Getting Started

### Prerequisites

* Node.js `>= 16.0.0`
* Active MongoDB connection string (Local or MongoDB Atlas)

### Installation

```bash
# Clone the repository
git clone https://github.com/fabogit/interview-tickets-ticketing-server.git
cd interview-tickets-ticketing-server

# Install dependencies
npm install
```

### Environment Configuration

Create a `.env` file in the project root:

```env
PORT=3000
MONGODB_URI="mongodb://localhost:27017/tickets_db"
JWT_SECRET="your-super-secret-jwt-key"
```

### Running the Application

```bash
# Start server with live reload via nodemon
npm start
```

---

## 📄 License & Author

Developed by [@fabogit](https://github.com/fabogit).
