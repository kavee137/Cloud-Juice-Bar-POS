# Mini E-Commerce Microservices (ECA Final Project)

This repository contains a simple microservice-based system designed to match the module requirements:

- Spring Cloud platform: Config Server, Eureka, API Gateway
- 3 business microservices: User (Relational DB), Product (NoSQL + Cloud Storage), Order (NoSQL)
- Frontend: simple static HTML/JS that calls the API Gateway
- Deployment expectation: non-containerized VMs on GCP managed by PM2

Directory layout:

- `platform/config-server` Spring Cloud Config Server
- `platform/eureka-server` Eureka service registry
- `platform/api-gateway` Spring Cloud Gateway
- `services/user-service` User management (MySQL/PostgreSQL via JPA)
- `services/product-service` Product + image upload (MongoDB + Storage provider)
- `services/order-service` Orders (MongoDB)
- `config-repo` Config Server backing repository (YAML)
- `frontend` Static frontend
- `pm2` PM2 ecosystem config for VM deployments

## Prereqs

- Java: target is Java 25 (set in Maven). Install JDK 25 on your machine/VMs.
- Maven 3.9+
- Databases (for local):
  - MySQL or PostgreSQL for `user-service`
  - MongoDB for `product-service` and `order-service`

## Local run (dev)

1) Start platform services:

```bash
mvn -f platform/config-server/pom.xml spring-boot:run
mvn -f platform/eureka-server/pom.xml spring-boot:run
mvn -f platform/api-gateway/pom.xml spring-boot:run
```

2) Start microservices:

```bash
mvn -f services/user-service/pom.xml spring-boot:run
mvn -f services/product-service/pom.xml spring-boot:run
mvn -f services/order-service/pom.xml spring-boot:run
```

3) Open:

- Eureka dashboard: `http://localhost:8761`
- Gateway: `http://localhost:8080`
- Frontend: open `frontend/index.html`

## Health checks (LB / MIG)

All services expose Actuator health endpoints.

- Basic: `GET /actuator/health`
- Readiness (recommended for LB): `GET /actuator/health/readiness`
- Liveness (recommended for autohealing): `GET /actuator/health/liveness`

## VM deployment checklist (no containers)

This matches the lecture pattern: build fat JARs, run with PM2 on Compute Engine VMs.

### 1) Build JARs (on your machine/CI)

```bash
mvn -f platform/config-server/pom.xml -DskipTests package
mvn -f platform/eureka-server/pom.xml -DskipTests package
mvn -f platform/api-gateway/pom.xml -DskipTests package
mvn -f services/user-service/pom.xml -DskipTests package
mvn -f services/product-service/pom.xml -DskipTests package
mvn -f services/order-service/pom.xml -DskipTests package
```

### 2) Upload code/JARs to the VM image

Typical layout on VM: `/opt/app` containing this repo.

### 3) Install runtime on the VM

- JDK 25
- Node.js + PM2

### 4) Set required env vars (example)

```bash
export CONFIG_REPO_PATH=/opt/app/config-repo
export CONFIG_SERVER_URL=http://127.0.0.1:8888
export EUREKA_URL=http://127.0.0.1:8761/eureka

# User DB (Cloud SQL)
export USER_DB_URL='jdbc:postgresql://<CLOUD_SQL_PRIVATE_IP>:5432/eca_users'
export USER_DB_USERNAME='...'
export USER_DB_PASSWORD='...'

# Mongo (Mongo VM / Atlas / etc)
export PRODUCT_MONGO_URI='mongodb://<MONGO_HOST>:27017/eca_products'
export ORDER_MONGO_URI='mongodb://<MONGO_HOST>:27017/eca_orders'

# Storage
export STORAGE_PROVIDER=gcs
export GCS_PROJECT_ID='CHANGE_ME'
export GCS_BUCKET='CHANGE_ME'
export GCS_CREDENTIALS_PATH='/opt/app/keys/service-account.json'
```

Template you can copy/edit on the VM: `docs/env-template.sh`

### 5) Start with PM2

```bash
pm2 start pm2/ecosystem.config.js
pm2 monit
pm2 save
pm2 startup
```

### 6) GCP infra (minimum mapping)

- Instance templates + Managed Instance Groups (2+ instances each)
- Load Balancer + health checks in front of the API Gateway
- Cloud DNS pointing to the LB
- Firewall rules allow LB -> gateway + internal service ports

More detailed checklist: `docs/gcp-deploy.md`


## Build JARs

```bash
mvn -f platform/config-server/pom.xml -DskipTests package
mvn -f platform/eureka-server/pom.xml -DskipTests package
mvn -f platform/api-gateway/pom.xml -DskipTests package
mvn -f services/user-service/pom.xml -DskipTests package
mvn -f services/product-service/pom.xml -DskipTests package
mvn -f services/order-service/pom.xml -DskipTests package
```

## Notes

- `product-service` supports `storage.provider=local` (default) and `storage.provider=gcs`.
- GCP identifiers/paths are placeholders in `config-repo/*.yml`. Replace with your real values.
