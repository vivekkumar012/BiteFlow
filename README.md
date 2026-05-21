# 🍔 BiteFlow — Zomato Clone with Microservices

> A full-stack food delivery platform built with Node.js microservices, RabbitMQ, Socket.IO, and deployed on Render + Vercel.

---

## 📌 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Microservices](#microservices)
- [Tech Stack](#tech-stack)
- [Features](#features)
- [Getting Started](#getting-started)
- [Environment Variables](#environment-variables)
- [Docker Setup](#docker-setup)
- [Deployment](#deployment)
- [Roles & Permissions](#roles--permissions)
- [Payment Gateways](#payment-gateways)
- [Real-Time Features](#real-time-features)
- [Project Structure](#project-structure)

---

## Overview

**BiteFlow** is a production-grade food delivery application inspired by Zomato. It is built using a microservices architecture where each service is independently deployable, scalable, and communicates asynchronously via RabbitMQ.

The platform supports four user roles — **Customer**, **Restaurant (Seller)**, **Delivery Partner (Rider)**, and **Admin** — with real-time order tracking, live rider location, and dual payment gateway support.

---

## Architecture

```
                        ┌─────────────────────┐
                        │    API Gateway /     │
                        │    Client (Vercel)   │
                        └────────┬────────────┘
                                 │
          ┌──────────────────────┼──────────────────────┐
          │                      │                      │
   ┌──────▼──────┐      ┌────────▼──────┐      ┌───────▼───────┐
   │ Auth Service│      │  Restaurant   │      │ Rider Service │
   │             │      │   Service     │      │               │
   └──────┬──────┘      └────────┬──────┘      └───────┬───────┘
          │                      │                      │
          └──────────────────────▼──────────────────────┘
                                 │
                        ┌────────▼────────┐
                        │    RabbitMQ     │  ← Message Broker (AWS/Docker)
                        └────────┬────────┘
                                 │
          ┌──────────────────────┼──────────────────────┐
          │                      │                      │
   ┌──────▼──────┐      ┌────────▼──────┐      ┌───────▼───────┐
   │ Utils Service│     │ Admin Service │      │Realtime Service│
   │ (Files+Pay) │      │               │      │  (Socket.IO)  │
   └─────────────┘      └───────────────┘      └───────────────┘
```

---

## Microservices

| Service | Responsibility |
|---|---|
| **Auth Service** | Registration, login, JWT authentication |
| **Restaurant Service** | Restaurant CRUD, menu management, cart, orders, addresses |
| **Rider Service** | Rider profile, order acceptance, delivery management |
| **Admin Service** | Validate and approve restaurants and riders |
| **Realtime Service** | Socket.IO hub — live order status, rider location tracking |
| **Utils Service** | File uploads (Cloudinary/S3), Razorpay & Stripe payment processing |

---

## Tech Stack

**Backend**
- Node.js + Express.js
- RabbitMQ (message broker)
- MongoDB + Mongoose
- JWT (authentication)
- Socket.IO (real-time)
- Docker + Docker Compose

**Frontend**
- React.js / Next.js
- Leaflet.js / Google Maps (rider tracking)
- Deployed on Vercel

**Payments**
- Razorpay (India)
- Stripe (Global)

**Storage & Media**
- Cloudinary or AWS S3 (file uploads)

**Infrastructure**
- Docker (containerization)
- Render (backend deployment)
- AWS EC2 / CloudAMQP (RabbitMQ hosting)

---

## Features

### Customer
- Browse restaurants and menus
- Add items to cart, place orders
- Pay via Razorpay or Stripe
- Real-time order status updates
- Live rider location on map
- Sound notifications on order updates

### Restaurant (Seller)
- Register and manage restaurant profile
- Open/close restaurant toggle
- Add, update, and remove menu items
- Manage incoming orders
- Manage delivery addresses

### Rider (Delivery Partner)
- Create and manage rider profile
- Accept or reject delivery requests
- Navigate to delivery location (map integration)
- Real-time location broadcasting

### Admin
- Validate restaurant registrations
- Validate and approve rider accounts
- Manage platform operations

---

## Getting Started

### Prerequisites

- Node.js >= 18
- Docker & Docker Compose
- MongoDB (local or Atlas)
- RabbitMQ (local or CloudAMQP)

### Clone the Repository

```bash
git clone https://github.com/yourusername/biteflow.git
cd biteflow
```

### Install Dependencies (per service)

```bash
cd auth-service && npm install
cd ../restaurant-service && npm install
cd ../rider-service && npm install
cd ../admin-service && npm install
cd ../realtime-service && npm install
cd ../utils-service && npm install
```

### Run All Services (Development)

```bash
# From root
docker-compose up --build
```

Or run individually:

```bash
cd auth-service && npm run dev
```

---

## Environment Variables

Each service has its own `.env` file. Below are the common variables:

### auth-service/.env
```env
PORT=4001
MONGO_URI=mongodb://localhost:27017/biteflow-auth
JWT_SECRET=your_jwt_secret
RABBITMQ_URL=amqp://localhost
```

### restaurant-service/.env
```env
PORT=4002
MONGO_URI=mongodb://localhost:27017/biteflow-restaurant
JWT_SECRET=your_jwt_secret
RABBITMQ_URL=amqp://localhost
```

### rider-service/.env
```env
PORT=4003
MONGO_URI=mongodb://localhost:27017/biteflow-rider
JWT_SECRET=your_jwt_secret
RABBITMQ_URL=amqp://localhost
```

### admin-service/.env
```env
PORT=4004
MONGO_URI=mongodb://localhost:27017/biteflow-admin
JWT_SECRET=your_jwt_secret
RABBITMQ_URL=amqp://localhost
```

### realtime-service/.env
```env
PORT=4005
RABBITMQ_URL=amqp://localhost
```

### utils-service/.env
```env
PORT=4006
MONGO_URI=mongodb://localhost:27017/biteflow-utils
RABBITMQ_URL=amqp://localhost
CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret
RAZORPAY_KEY_ID=your_razorpay_key
RAZORPAY_KEY_SECRET=your_razorpay_secret
STRIPE_SECRET_KEY=your_stripe_secret
```

---

## Docker Setup

### docker-compose.yml (root)

```yaml
version: "3.8"

services:
  rabbitmq:
    image: rabbitmq:3-management
    ports:
      - "5672:5672"
      - "15672:15672"

  auth-service:
    build: ./auth-service
    ports:
      - "4001:4001"
    env_file: ./auth-service/.env
    depends_on:
      - rabbitmq

  restaurant-service:
    build: ./restaurant-service
    ports:
      - "4002:4002"
    env_file: ./restaurant-service/.env
    depends_on:
      - rabbitmq

  rider-service:
    build: ./rider-service
    ports:
      - "4003:4003"
    env_file: ./rider-service/.env
    depends_on:
      - rabbitmq

  admin-service:
    build: ./admin-service
    ports:
      - "4004:4004"
    env_file: ./admin-service/.env
    depends_on:
      - rabbitmq

  realtime-service:
    build: ./realtime-service
    ports:
      - "4005:4005"
    env_file: ./realtime-service/.env
    depends_on:
      - rabbitmq

  utils-service:
    build: ./utils-service
    ports:
      - "4006:4006"
    env_file: ./utils-service/.env
    depends_on:
      - rabbitmq
```

### Dockerfile (per service)

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 4001
CMD ["npm", "start"]
```

---

## Deployment

### Backend — Render

1. Push each service to its own GitHub repository (or as separate Render services from a monorepo).
2. Create a new **Web Service** on [render.com](https://render.com) for each microservice.
3. Set root directory to the service folder (e.g., `auth-service`).
4. Add environment variables in the Render dashboard.
5. Deploy — Render auto-detects the Dockerfile.

### RabbitMQ — CloudAMQP (AWS)

1. Create a free instance on [cloudamqp.com](https://www.cloudamqp.com).
2. Copy the AMQP URL and set it as `RABBITMQ_URL` in all service `.env` files.

### Frontend — Vercel

```bash
cd frontend
vercel deploy
```

Set `NEXT_PUBLIC_API_URL` to point to your Render backend services.

---

## Roles & Permissions

| Action | Customer | Restaurant | Rider | Admin |
|---|:---:|:---:|:---:|:---:|
| Browse restaurants | ✅ | ✅ | ✅ | ✅ |
| Place orders | ✅ | ❌ | ❌ | ❌ |
| Manage restaurant | ❌ | ✅ | ❌ | ❌ |
| Accept deliveries | ❌ | ❌ | ✅ | ❌ |
| Validate accounts | ❌ | ❌ | ❌ | ✅ |
| View live tracking | ✅ | ❌ | ✅ | ❌ |

---

## Payment Gateways

### Razorpay (India)
- Used for Indian customers
- Webhook-based order confirmation
- Handled in `utils-service`

### Stripe (Global)
- Used for international customers
- Supports cards, wallets
- Handled in `utils-service`

Both gateways emit a `payment.success` or `payment.failed` event via RabbitMQ to update order status across services.

---

## Real-Time Features

Powered by **Socket.IO** in the `realtime-service`:

| Event | Description |
|---|---|
| `order:placed` | Notifies restaurant of new order |
| `order:accepted` | Notifies customer that restaurant confirmed |
| `order:picked` | Notifies customer that rider picked up order |
| `order:delivered` | Final delivery confirmation |
| `rider:location` | Broadcasts rider GPS coordinates to customer |
| `notification:sound` | Triggers sound alert on relevant dashboard |

Customers can see the rider's live location on an embedded map (Leaflet.js / Google Maps) that updates via `rider:location` events.

---

## Project Structure

```
biteflow/
├── auth-service/
│   ├── src/
│   │   ├── controllers/
│   │   ├── models/
│   │   ├── routes/
│   │   ├── middleware/
│   │   └── rabbitmq/
│   ├── Dockerfile
│   └── package.json
│
├── restaurant-service/
│   ├── src/
│   │   ├── controllers/
│   │   ├── models/
│   │   ├── routes/
│   │   └── rabbitmq/
│   ├── Dockerfile
│   └── package.json
│
├── rider-service/
│   ├── src/
│   └── Dockerfile
│
├── admin-service/
│   ├── src/
│   └── Dockerfile
│
├── realtime-service/
│   ├── src/
│   │   └── socket/
│   └── Dockerfile
│
├── utils-service/
│   ├── src/
│   │   ├── upload/
│   │   └── payment/
│   └── Dockerfile
│
├── frontend/
│   ├── pages/
│   ├── components/
│   └── public/
│
├── docker-compose.yml
└── README.md
```

---

## Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/your-feature`)
3. Commit your changes (`git commit -m 'Add your feature'`)
4. Push to the branch (`git push origin feature/your-feature`)
5. Open a Pull Request

---

## License

This project is licensed under the MIT License.

---

> Built with ❤️ to learn how real food delivery platforms work under the hood.