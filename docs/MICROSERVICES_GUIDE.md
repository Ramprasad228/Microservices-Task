# Microservices Task Guide

## 1. Project Overview

This project demonstrates a small microservices application built with Node.js, Express, Docker, and Docker Compose.

The application contains four services:

- **User Service**: provides user data.
- **Product Service**: provides product data.
- **Order Service**: creates and lists orders.
- **Gateway Service**: provides a single API entry point and forwards requests to the other services.

Each service runs in its own container and has its own Dockerfile.

## 2. Repository Structure

```text
Microservices-Task/
├── compose.yaml
├── .dockerignore
├── README.md
├── docs/
│   └── MICROSERVICES_GUIDE.md
└── Microservices/
    ├── gateway-service/
    │   ├── Dockerfile
    │   ├── app.js
    │   └── package.json
    ├── order-service/
    │   ├── Dockerfile
    │   ├── app.js
    │   └── package.json
    ├── product-service/
    │   ├── Dockerfile
    │   ├── app.js
    │   └── package.json
    └── user-service/
        ├── Dockerfile
        ├── app.js
        └── package.json
```

## 3. Service Architecture

Docker Compose creates a shared network for the services. The gateway communicates with the backend services using their Compose service names:

```text
Client
  |
  v
Gateway Service :3003
  |-- user-service    :3000
  |-- product-service :3001
  `-- order-service   :3002
```

The service names are resolved automatically by Docker Compose. For example, the gateway calls `http://user-service:3000/users` from inside the Compose network.

## 4. Docker Configuration

Each service has an independent Dockerfile with the same basic build process:

1. Start from the Node.js 20 Alpine image.
2. Set `/app` as the working directory.
3. Copy the service package files.
4. Install production dependencies.
5. Copy the service source code.
6. Start `app.js`.

The Compose file uses each service directory as its build context. This keeps each image self-contained and allows the services to be built independently.

## 5. Prerequisites

Install and start Docker Desktop with Docker Compose support enabled.

Verify Docker is available:

```bash
docker --version
docker compose version
```

## 6. Build and Start the Application

Run these commands from the repository root:

```bash
docker compose up --build
```

To run the services in the background:

```bash
docker compose up --build -d
```

The published ports are:

| Service | Container Port | Host Port |
| --- | ---: | ---: |
| User Service | 3000 | 3000 |
| Product Service | 3001 | 3001 |
| Order Service | 3002 | 3002 |
| Gateway Service | 3003 | 3003 |

## 7. Verify the Containers

Check the current container status:

```bash
docker compose ps
```

All four services should show a status of `Up` and the expected port mappings.

Check stopped containers as well:

```bash
docker compose ps -a
```

View service logs:

```bash
docker compose logs -f
```

View logs for one service only:

```bash
docker compose logs -f gateway-service
```

## 8. Health Checks

Each service exposes a `/health` endpoint:

```bash
curl http://localhost:3000/health
curl http://localhost:3001/health
curl http://localhost:3002/health
curl http://localhost:3003/health
```

Expected responses are JSON objects showing that the corresponding service is healthy.

## 9. API Endpoints

### User Service

```bash
curl http://localhost:3000/users
```

### Product Service

```bash
curl http://localhost:3001/products
```

### Order Service

List orders:

```bash
curl http://localhost:3002/orders
```

Create an order:

```bash
curl -X POST http://localhost:3002/orders \
  -H "Content-Type: application/json" \
  -d '{"userId":1,"productId":1}'
```

### Gateway Service

The gateway exposes the backend functionality through `/api` routes:

```bash
curl http://localhost:3003/api/users
curl http://localhost:3003/api/products
curl http://localhost:3003/api/orders
```

Create an order through the gateway:

```bash
curl -X POST http://localhost:3003/api/orders \
  -H "Content-Type: application/json" \
  -d '{"userId":1,"productId":1}'
```

## 10. Stopping and Cleaning Up

Stop the running containers:

```bash
docker compose down
```

Stop the containers and remove their images:

```bash
docker compose down --rmi local
```

## 11. Troubleshooting

### Curl cannot connect to localhost

If curl reports `Failed to connect`, check whether the containers are running:

```bash
docker compose ps
```

If the services are stopped, start them:

```bash
docker compose up -d
```

### A container exits immediately

Inspect its logs:

```bash
docker compose logs user-service
docker compose logs product-service
docker compose logs order-service
docker compose logs gateway-service
```

The exit code `137` commonly indicates that the container was terminated by the operating system or Docker due to resource pressure. Check Docker Desktop resource settings and restart the stack if necessary.

### Rebuild after source changes

```bash
docker compose up --build -d
```

## 12. Current Limitations

- Service data is stored in memory and is lost when the order container restarts.
- The Compose `depends_on` settings control startup order but do not wait for application health checks.
- Authentication and authorization are not implemented.
- The services use development-level sample data and should not be treated as production services without additional security, persistence, and observability features.
