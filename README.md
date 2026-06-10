# Event-Driven E-commerce

NestJS microservices project with Kafka in KRaft mode. The current version includes the first authentication flows in `auth-service`; the other services are boilerplates with health checks, Swagger, and demo Kafka producers/consumers.

## Services

- `auth-service`: <http://localhost:3001>
- `orders-service`: <http://localhost:3002>
- `inventory-service`: <http://localhost:3003>
- `shipping-service`: <http://localhost:3004>
- `notification-service`: <http://localhost:3005>
- `customer-service`: <http://localhost:3006>
- `seller-service`: <http://localhost:3007>
- `catalog-service`: <http://localhost:3008>
- `search-service`: <http://localhost:3009>
- `recommendation-service`: <http://localhost:3010>
- `cart-service`: <http://localhost:3011>
- `promotion-service`: <http://localhost:3012>
- `payment-service`: <http://localhost:3013>
- `return-service`: <http://localhost:3014>
- `review-service`: <http://localhost:3015>
- `question-service`: <http://localhost:3016>
- `settlement-service`: <http://localhost:3017>
- `support-service`: <http://localhost:3018>
- `admin-service`: <http://localhost:3019>
- `analytics-service`: <http://localhost:3020>

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
curl http://localhost:3006/health
curl http://localhost:3007/health
curl http://localhost:3008/health
curl http://localhost:3009/health
curl http://localhost:3010/health
curl http://localhost:3011/health
curl http://localhost:3012/health
curl http://localhost:3013/health
curl http://localhost:3014/health
curl http://localhost:3015/health
curl http://localhost:3016/health
curl http://localhost:3017/health
curl http://localhost:3018/health
curl http://localhost:3019/health
curl http://localhost:3020/health
```

## Swagger

- `auth-service`: <http://localhost:3001/docs>
- `orders-service`: <http://localhost:3002/docs>
- `inventory-service`: <http://localhost:3003/docs>
- `shipping-service`: <http://localhost:3004/docs>
- `notification-service`: <http://localhost:3005/docs>
- `customer-service`: <http://localhost:3006/docs>
- `seller-service`: <http://localhost:3007/docs>
- `catalog-service`: <http://localhost:3008/docs>
- `search-service`: <http://localhost:3009/docs>
- `recommendation-service`: <http://localhost:3010/docs>
- `cart-service`: <http://localhost:3011/docs>
- `promotion-service`: <http://localhost:3012/docs>
- `payment-service`: <http://localhost:3013/docs>
- `return-service`: <http://localhost:3014/docs>
- `review-service`: <http://localhost:3015/docs>
- `question-service`: <http://localhost:3016/docs>
- `settlement-service`: <http://localhost:3017/docs>
- `support-service`: <http://localhost:3018/docs>
- `admin-service`: <http://localhost:3019/docs>
- `analytics-service`: <http://localhost:3020/docs>

## Demo Events

```bash
curl -X POST http://localhost:3001/events/demo
curl -X POST http://localhost:3002/events/demo
curl -X POST http://localhost:3003/events/demo
curl -X POST http://localhost:3004/events/demo
curl -X POST http://localhost:3005/events/demo
curl -X POST http://localhost:3006/events/demo
curl -X POST http://localhost:3007/events/demo
curl -X POST http://localhost:3008/events/demo
curl -X POST http://localhost:3009/events/demo
curl -X POST http://localhost:3010/events/demo
curl -X POST http://localhost:3011/events/demo
curl -X POST http://localhost:3012/events/demo
curl -X POST http://localhost:3013/events/demo
curl -X POST http://localhost:3014/events/demo
curl -X POST http://localhost:3015/events/demo
curl -X POST http://localhost:3016/events/demo
curl -X POST http://localhost:3017/events/demo
curl -X POST http://localhost:3018/events/demo
curl -X POST http://localhost:3019/events/demo
curl -X POST http://localhost:3020/events/demo
```

Each service publishes to its own demo topic and also has a consumer for the same topic with a basic `console.log`.
