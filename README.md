# Simple E-Commerce Microservices Learning Project

This project is a simple, understandable implementation of an e-commerce application built using a microservices architecture. It is designed for learning purposes, specifically focusing on Node.js, Docker, MongoDB, and Kubernetes.

## Project Architecture

The application consists of a React frontend and four Node.js backend microservices.

```text
                  ┌──────────────┐
                  │    React     │
                  │    Vercel    │
                  └──────┬───────┘
                         │
                         ▼
                Kubernetes Ingress
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
      Auth           Product            Cart
     Service         Service           Service
        │                │                │
        └────────────────┼────────────────┘
                         │
                         ▼
                    Order Service
                         │
                         ▼
                      MongoDB
```

## Folder Structure

```text
├── services/
│   ├── auth-service/        # Authentication (Demo)
│   ├── product-service/     # Product catalog
│   ├── cart-service/        # Shopping cart
│   └── order-service/       # Order processing
├── frontend/                # React (Vite + Tailwind)
└── docker-compose.yml       # Orchestration for backend services
```

## Services & Ports

| Service | Port | Description |
|---------|------|-------------|
| Auth Service | 3001 | Simple demo authentication |
| Product Service | 3002 | Product management and listing |
| Cart Service | 3003 | User shopping cart |
| Order Service | 3004 | Order creation and history |
| MongoDB | 27017 | Database |

## How to Start

### 1. MongoDB & Backend Services (Docker Compose)
Run the following command to start the database and all microservices:
```bash
docker-compose up --build
```

### 2. React Frontend
Navigate to the frontend directory and start the development server:
```bash
cd frontend
npm install
npm run dev
```

## Environment Variables (Frontend)

The frontend uses the following environment variables to communicate with the backend:

- `VITE_AUTH_SERVICE_URL`
- `VITE_PRODUCT_SERVICE_URL`
- `VITE_CART_SERVICE_URL`
- `VITE_ORDER_SERVICE_URL`

## Kubernetes Readiness

All backend services are designed to be Kubernetes-ready:
- Listen on `0.0.0.0`.
- Configurable ports.
- Include a `/health` endpoint returning `200 OK`.
- Stateless and containerized.

## Disclaimer

**The authentication implementation is for learning/demo purposes only and is NOT production secure.**
</task_progress>
</write_to_file>