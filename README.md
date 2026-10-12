# 🛍️ NEXUS — E-Commerce Microservices Platform
### Built with Docker & Kubernetes | Node.js | MongoDB | REST API

![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-47A248?style=for-the-badge&logo=mongodb&logoColor=white)
![Express](https://img.shields.io/badge/Express-000000?style=for-the-badge&logo=express&logoColor=white)

---

## 📌 Project Overview

NEXUS is a **production-grade E-Commerce platform** built using **Microservices Architecture**. The system is divided into 5 independent services — each running in its own Docker container with its own database. The platform supports user authentication, product management, order processing, and real-time notifications.

This project demonstrates:
-  Microservice separation and independent deployment
-  Docker containerization with production best practices
-  Kubernetes orchestration with health checks, service discovery, and resource limits
-  Fault tolerance — one service failing does NOT affect others
-  REST API communication between services

---

## 🏗️ Architecture Diagram

```
                        ┌─────────────────────────────────────┐
                        │           CLIENT / BROWSER           │
                        │        http://localhost:8080         │
                        └──────────────┬──────────────────────┘
                                       │
                                       ▼
                        ┌─────────────────────────────────────┐
                        │           API GATEWAY               │
                        │         Port: 3000 (NodePort)       │
                        │   • Rate Limiting (100 req/15min)   │
                        │   • CORS Handling                   │
                        │   • Request Routing                 │
                        └──────┬───────┬──────┬──────┬───────┘
                               │       │      │      │
               ┌───────────────┘       │      │      └──────────────────┐
               │                       │      │                         │
               ▼                       ▼      ▼                         ▼
  ┌─────────────────────┐  ┌──────────────────┐  ┌──────────────┐  ┌────────────────────┐
  │    USER SERVICE     │  │ PRODUCT SERVICE  │  │ORDER SERVICE │  │NOTIFICATION SERVICE│
  │     Port: 3001      │  │   Port: 3002     │  │  Port: 3003  │  │     Port: 3004     │
  │  • Register         │  │  • List Products │  │ • Place Order│  │  • Send Alerts     │
  │  • Login (JWT)      │  │  • Add Product   │  │ • Get Orders │  │  • Order Alerts    │
  │  • Profile          │  │  • Update Stock  │  │ • Update     │  │  • In-memory store │
  └────────┬────────────┘  └────────┬─────────┘  └──────┬───────┘  └────────────────────┘
           │                        │                    │
           ▼                        ▼                    ▼
  ┌─────────────────┐    ┌──────────────────┐  ┌──────────────────┐
  │  MongoDB:27017  │    │  MongoDB:27017   │  │  MongoDB:27017   │
  │  DB: users      │    │  DB: products    │  │  DB: orders      │
  └─────────────────┘    └──────────────────┘  └──────────────────┘

  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
                    KUBERNETES LAYER (Minikube)
  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Each service → 1 Deployment + 1 Service + Resource Limits
  user-service → 2 Replicas (High Availability)
  API Gateway  → NodePort (External Access)
  Databases    → ClusterIP (Internal Only - Secure)
```

---

## 🗂️ Project Structure

```
E-Commerce-Microservices/
├── api-gateway/
│   ├── index.js          # Gateway routing + rate limiting + CORS
│   ├── Dockerfile        # Production Dockerfile
│   ├── package.json
│   └── .dockerignore
├── user-service/
│   ├── index.js          # Register, Login, JWT auth, Profile
│   ├── Dockerfile
│   ├── package.json
│   └── .dockerignore
├── product-service/
│   ├── index.js          # CRUD products, stock management
│   ├── Dockerfile
│   ├── package.json
│   └── .dockerignore
├── order-service/
│   ├── index.js          # Place orders, inter-service calls
│   ├── Dockerfile
│   ├── package.json
│   └── .dockerignore
├── notification-service/
│   ├── index.js          # Send + store notifications
│   ├── Dockerfile
│   ├── package.json
│   └── .dockerignore
├── frontend/
│   └── index.html        # Full e-commerce UI (NEXUS)
├── k8s/
│   ├── namespace.yaml
│   ├── api-gateway.yaml
│   ├── user-service.yaml
│   ├── product-service.yaml
│   ├── order-service-deployment.yaml
│   ├── notification-service-deployment.yaml
│   ├── mongo-users.yaml
│   ├── mongo-products.yaml
│   └── mongo-orders.yaml
├── docker-compose.yml
└── README.md
```

---

## 🔧 Services & Responsibilities

| Service | Port | Technology | Database | Responsibility |
|---------|------|------------|----------|----------------|
| API Gateway | 3000 | Express + http-proxy-middleware | None | Routing, Rate limiting, CORS |
| User Service | 3001 | Express + JWT + bcrypt | MongoDB (users) | Registration, Login, Profiles |
| Product Service | 3002 | Express + Mongoose | MongoDB (products) | Catalog, Stock management |
| Order Service | 3003 | Express + Axios | MongoDB (orders) | Orders, Inter-service calls |
| Notification Service | 3004 | Express | In-memory | Alerts, Notifications |

---

## 📡 API Documentation

### 🔐 User Service — `/api/users`

| Method | Endpoint | Description | Request Body |
|--------|----------|-------------|--------------|
| POST | `/api/users/register` | Register new user | `{name, email, password}` |
| POST | `/api/users/login` | Login, returns JWT token | `{email, password}` |
| GET | `/api/users/profile/:id` | Get user profile | — |

**Register Example:**
```bash
curl -X POST http://localhost:3000/api/users/register \
  -H "Content-Type: application/json" \
  -d '{"name":"John Doe","email":"john@test.com","password":"password123"}'
```

**Response:**
```json
{
  "id": "64abc123...",
  "name": "John Doe",
  "email": "john@test.com"
}
```

---

### 📦 Product Service — `/api/products`

| Method | Endpoint | Description | Request Body |
|--------|----------|-------------|--------------|
| GET | `/api/products` | List all products | — |
| GET | `/api/products/:id` | Get single product | — |
| POST | `/api/products` | Create product | `{name, description, price, stock, category}` |
| PATCH | `/api/products/:id/stock` | Update stock | `{quantity}` |

**Create Product Example:**
```bash
curl -X POST http://localhost:3000/api/products \
  -H "Content-Type: application/json" \
  -d '{"name":"MacBook Pro","description":"Apple Silicon","price":2499.99,"stock":10,"category":"Electronics"}'
```

---

### 🛒 Order Service — `/api/orders`

| Method | Endpoint | Description | Request Body |
|--------|----------|-------------|--------------|
| POST | `/api/orders` | Place new order | `{userId, items:[{productId, quantity}]}` |
| GET | `/api/orders/user/:userId` | Get user orders | — |
| PATCH | `/api/orders/:id/status` | Update order status | `{status}` |

**Place Order Example:**
```bash
curl -X POST http://localhost:3000/api/orders \
  -H "Content-Type: application/json" \
  -d '{"userId":"64abc...","items":[{"productId":"64xyz...","quantity":2}]}'
```

---

### 🔔 Notification Service — `/api/notifications`

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/notifications/notify` | Send notification |
| GET | `/api/notifications/user/:userId` | Get user notifications |

---

## 🐳 Part 1 — Docker Deployment

### Prerequisites
- Docker Desktop installed
- Node.js v20+

### Quick Start

```bash
# Clone the repository
git clone https://github.com/YOUR_USERNAME/ecommerce-microservices.git
cd ecommerce-microservices

# Start all services
docker compose up -d

# Check all containers running
docker compose ps

# Test gateway
curl http://localhost:3000/health
```

### View Running Containers
```bash
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

### View Database Data
```bash
# See all users
docker exec -it mongo-users mongosh users --eval "db.users.find({},{password:0}).forEach(printjson)"

# See all products
docker exec -it mongo-products mongosh products --eval "db.products.find({}).forEach(printjson)"

# See all orders
docker exec -it mongo-orders mongosh orders --eval "db.orders.find({}).forEach(printjson)"
```

### Stop All Services
```bash
docker compose down
```

---

## ☸️ Part 2 — Kubernetes Deployment

### Prerequisites
- Docker Desktop
- Minikube
- kubectl

### Step-by-Step Deployment

```bash
# Step 1 — Start Minikube
minikube start --driver=docker --memory=4096 --cpus=2

# Step 2 — Point Docker to Minikube
eval $(minikube docker-env)

# Step 3 — Build images inside Minikube
docker compose build

# Step 4 — Apply all manifests
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/

# Step 5 — Watch pods start
kubectl get pods -n ecommerce --watch

# Step 6 — Get access URL
minikube service api-gateway -n ecommerce --url
```

### Verify Kubernetes Deployment

```bash
# All pods
kubectl get pods -n ecommerce

# All services (service discovery)
kubectl get services -n ecommerce

# All deployments
kubectl get deployments -n ecommerce

# Resource limits on a pod
kubectl describe pod -n ecommerce -l app=user-service | grep -A8 "Limits\|Requests"
```

### Test on Kubernetes
```bash
# Replace URL with minikube service URL
export K8S_URL=http://127.0.0.1:52853

curl $K8S_URL/health
curl -X POST $K8S_URL/api/users/register \
  -H "Content-Type: application/json" \
  -d '{"name":"K8s Test","email":"k8s@test.com","password":"test1234"}'
curl $K8S_URL/api/products
```

---

## 🛡️ Fault Tolerance Demo

### Test 1 — Kill Notification Service, Orders Still Work
```bash
docker stop notification-service
curl -X POST http://localhost:3000/api/orders \
  -H "Content-Type: application/json" \
  -d '{"userId":"USER_ID","items":[{"productId":"PRODUCT_ID","quantity":1}]}'
docker start notification-service
# ✅ Order placed successfully even without notification service!
```

### Test 2 — Kill Product Service, Login Still Works
```bash
docker stop product-service
curl -X POST http://localhost:3000/api/users/login \
  -H "Content-Type: application/json" \
  -d '{"email":"john@test.com","password":"password123"}'
docker start product-service
# ✅ Login works even without product service!
```

### Test 3 — Kubernetes Auto-Restart
```bash
# Delete a pod — Kubernetes automatically restarts it!
kubectl delete pod -n ecommerce -l app=notification-service
kubectl get pods -n ecommerce -w
# ✅ New pod created automatically within seconds!
```

---

## ⚙️ Kubernetes Configuration Details

### Resource Limits (Every Pod)
```yaml
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
  limits:
    memory: "256Mi"
    cpu: "200m"
```

### Health Checks (Readiness + Liveness Probes)
```yaml
readinessProbe:
  httpGet:
    path: /health
    port: 3001
  initialDelaySeconds: 10
  periodSeconds: 5
livenessProbe:
  httpGet:
    path: /health
    port: 3001
  initialDelaySeconds: 15
  periodSeconds: 10
```

### Service Discovery
```
Services communicate by name — no IP needed!
order-service  → http://product-service:3002
order-service  → http://notification-service:3004
all services   → mongodb://mongo-users:27017
```

### High Availability
```
user-service → replicas: 2 (always 2 pods running)
api-gateway  → NodePort (external access)
databases    → ClusterIP (internal only, secure)
```

---

## 🏭 Production-Grade Features

| Feature | Implementation |
|---------|---------------|
| Security | Non-root Docker containers (appuser) |
| Performance | Docker layer caching (package.json first) |
| Reliability | Health checks on every service |
| Scalability | Kubernetes with configurable replicas |
| Isolation | Each service has its own MongoDB database |
| Protection | Rate limiting — 100 requests per 15 minutes |
| Resilience | Fire-and-forget pattern for notifications |
| Persistence | Docker named volumes for all databases |

---

## 📚 Lessons Learned & Challenges Faced

### 1. Image Naming in Kubernetes
**Challenge:** Kubernetes manifest image names must exactly match Docker image names. Got `ErrImageNeverPull` errors because manifest said `ecommerce-microservices-user-service` but actual image was `e-commerce-microservices-system-docker-kubernetes--user-service`.

**Solution:** Used `sed` to fix all manifest files at once to match exact Docker image names.

### 2. Service Discovery
**Challenge:** Initially tried using IP addresses for inter-service communication which kept changing.

**Solution:** Used Docker/Kubernetes internal DNS — services find each other by name (`http://product-service:3002`). This works identically in both Docker Compose and Kubernetes.

### 3. Database Per Service
**Challenge:** Temptation to share one MongoDB for all services to save resources.

**Solution:** Kept strict database isolation — each service has its own MongoDB deployment. This ensures true microservice independence.

### 4. Readiness Probes Causing CrashLoopBackOff
**Challenge:** Order service kept restarting in Kubernetes even though logs showed it was working fine.

**Solution:** The readiness probe was checking `/health` before MongoDB connected. Removed probes temporarily to stabilize, then reimplemented with longer `initialDelaySeconds`.

### 5. Fault Tolerance Implementation
**Challenge:** When notification service was down, order placement was also failing.

**Solution:** Implemented fire-and-forget pattern using `.catch(() => {})` — order service sends notification but doesn't wait for response. If notification fails, order still succeeds.

### 6. CORS Issues with Frontend
**Challenge:** Frontend at `localhost:8080` couldn't call API at `localhost:3000` due to CORS policy.

**Solution:** Added `cors` middleware to API Gateway. Also fixed bug where `app.use(cors())` was called before `const app = express()`.

### 7. Docker Layer Caching
**Challenge:** Every code change triggered full `npm install` which took 2-3 minutes.

**Solution:** Copied `package.json` before source code in Dockerfile. Now npm install is cached and only runs when dependencies change.

---

## 🛠️ Technologies Used

| Technology | Purpose |
|------------|---------|
| Node.js v20 | Runtime for all microservices |
| Express.js | REST API framework |
| MongoDB | Database for each service |
| Mongoose | MongoDB ODM |
| JWT | Authentication tokens |
| bcryptjs | Password hashing |
| Axios | Inter-service HTTP calls |
| Docker | Containerization |
| Docker Compose | Multi-container orchestration (Development) |
| Kubernetes/Minikube | Container orchestration (Production) |
| http-proxy-middleware | API Gateway routing |
| express-rate-limit | Rate limiting security |

---

<div align="center">

**⭐ If you find this project interesting, consider starring the repo!**

Built by [Chetan Meshram](https://github.com/chetanmeshram10)

</div>

