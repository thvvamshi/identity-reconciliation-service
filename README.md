# Identity Reconciliation Service

A backend service that intelligently reconciles customer identities across multiple interactions by linking contact information based on shared email addresses and phone numbers.

The system maintains a unified customer profile while preserving historical records, ensuring that a single customer is represented consistently even when different contact details are used over time.

---

## Overview

Customer data often becomes fragmented when users interact with a platform using different combinations of emails and phone numbers. This project addresses that challenge by automatically identifying related contacts and consolidating them into a single logical identity.

The service exposes a REST API that accepts contact information and returns a unified customer view by applying identity resolution and contact-linking rules.

---

## Features

* Identity resolution using email and phone number matching
* Automatic contact linking
* Primary and secondary contact management
* Duplicate contact prevention
* Contact cluster merging
* Deterministic oldest-primary selection
* Email-only and phone-only support
* Transaction-safe database operations
* RESTful API architecture

---

## How It Works

### New Contact

If no existing contact matches the incoming information:

* A new primary contact is created.
* The contact becomes the root identity for future associations.

### Existing Contact

If the email or phone number already exists:

* Related contacts are discovered.
* The oldest primary contact is identified.
* New information is linked to the existing identity.

### Contact Merge

When two independent contact groups become connected:

* The oldest primary contact remains primary.
* The newer primary is converted into a secondary contact.
* All linked contacts are updated accordingly.

This ensures a single source of truth for each customer.

---

## Architecture

The application follows a clean MVC architecture with a repository layer.

```text
Client
   │
   ▼
Routes
   │
   ▼
Controller Layer
   │
   ▼
Business Logic
   │
   ▼
Repository Layer
   │
   ▼
MySQL Database
```

### Layers

#### Routes

Responsible for API endpoint registration.

#### Controllers

Handle request validation and response formatting.

#### Business Logic

Implements identity reconciliation rules and merge operations.

#### Repository

Manages all database interactions and transaction handling.

#### Database

Stores contacts and relationship metadata.

---

## Database Schema

```sql
CREATE TABLE Contact (
   id INT AUTO_INCREMENT PRIMARY KEY,
   phoneNumber VARCHAR(20),
   email VARCHAR(255),
   linkedId INT,
   linkPrecedence ENUM('primary', 'secondary') NOT NULL,
   createdAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
   updatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
   deletedAt TIMESTAMP NULL
);
```

### Contact Relationship Model

#### Primary Contact

Represents the canonical customer identity.

#### Secondary Contact

References an existing primary contact through `linkedId`.

This structure allows historical records to be preserved while presenting a unified customer profile.

---

## API Reference

### Identify Contact

```http
POST /identify
```

### Request Body

```json
{
  "email": "user@example.com",
  "phoneNumber": "9876543210"
}
```

Both fields are optional, but at least one must be provided.

---

### Response

```json
{
  "contact": {
    "primaryContatctId": 1,
    "emails": [
      "user@example.com"
    ],
    "phoneNumbers": [
      "9876543210"
    ],
    "secondaryContactIds": []
  }
}
```

---

## Example Workflow

### Request 1

```json
{
  "email": "alpha@gmail.com",
  "phoneNumber": "111"
}
```

Creates a new primary contact.

### Request 2

```json
{
  "email": "alpha@gmail.com",
  "phoneNumber": "222"
}
```

Creates a secondary contact linked to the existing primary.

### Request 3

```json
{
  "email": "beta@gmail.com",
  "phoneNumber": "222"
}
```

Links the new email to the same customer identity.

### Consolidated Response

```json
{
  "contact": {
    "primaryContatctId": 1,
    "emails": [
      "alpha@gmail.com",
      "beta@gmail.com"
    ],
    "phoneNumbers": [
      "111",
      "222"
    ],
    "secondaryContactIds": [2, 3]
  }
}
```

---

## Tech Stack

### Backend

* Node.js
* Express.js

### Database

* MySQL

### Design Principles

* MVC Architecture
* Repository Pattern
* Transaction Management
* RESTful API Design
* Environment-Based Configuration

---

## Project Structure

```text
├── controllers/
│   └── identityController.js
├── models/
│   └── contactRepository.js
├── routes/
│   └── identityRoutes.js
├── db.js
├── app.js
├── server.js
├── package.json
└── README.md
```

---

## Installation

### Clone Repository

```bash
git clone <repository-url>
cd identity-reconciliation-service
```

### Install Dependencies

```bash
npm install
```

### Configure Environment Variables

Create a `.env` file:

```env
DB_HOST=your_host
DB_USER=your_user
DB_PASS=your_password
DB_NAME=your_database
DB_PORT=your_port
PORT=3000
```

### Start Server

```bash
npm start
```

The application will be available at:

```text
http://localhost:3000
```

---

## Deployment

### Base URL

```text
https://bitespeed-identity-reconciliation-sy46.onrender.com
```

### Endpoint

```text
POST /identify
```

---

## Future Enhancements

* Contact graph visualization
* Identity merge audit logs
* Contact analytics dashboard
* Event-driven reconciliation pipeline
* Redis caching layer
* Monitoring and observability integration
* High-volume performance optimizations

---

## Key Learnings

This project demonstrates practical experience with:

* Relational database modeling
* Identity resolution systems
* Data consistency strategies
* Backend API development
* Transaction management
* Customer data reconciliation
* Scalable service design

---

## Author

**Vamshi Kumar**

Backend Developer | Node.js | Express.js | MySQL | System Design

LinkedIn: https://www.linkedin.com/in/bodavamshikumar
