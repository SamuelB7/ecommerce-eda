# Event-Driven E-commerce

NestJS microservices project with Kafka in KRaft mode. The current version includes the first authentication flows in `auth-service`; the other services are still initial boilerplates with demo Kafka producers and consumers.

## Services

- `auth-service`: <http://localhost:3001>
- `orders-service`: <http://localhost:3002>
- `inventory-service`: <http://localhost:3003>
- `shipping-service`: <http://localhost:3004>
- `notification-service`: <http://localhost:3005>

## Run Locally

```bash
docker compose up --build
```

Kafka runs in KRaft mode and may take a few seconds to register the broker. The microservices wait for the Kafka health check before starting.

If old containers from previous attempts remain:

```bash
docker compose down --remove-orphans
docker compose up --build
```

## Health Checks

```bash
curl http://localhost:3001/health
curl http://localhost:3002/health
curl http://localhost:3003/health
curl http://localhost:3004/health
curl http://localhost:3005/health
```

## Swagger

- `auth-service`: <http://localhost:3001/docs>
- `orders-service`: <http://localhost:3002/docs>
- `inventory-service`: <http://localhost:3003/docs>
- `shipping-service`: <http://localhost:3004/docs>
- `notification-service`: <http://localhost:3005/docs>

## Demo Events

```bash
curl -X POST http://localhost:3001/events/demo
curl -X POST http://localhost:3002/events/demo
curl -X POST http://localhost:3003/events/demo
curl -X POST http://localhost:3004/events/demo
curl -X POST http://localhost:3005/events/demo
```

Each service publishes to its own demo topic and also has a consumer for the same topic with a basic `console.log`.
