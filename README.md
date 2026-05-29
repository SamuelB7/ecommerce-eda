# Event-Driven E-commerce

Boilerplate inicial de microsservicos NestJS com Kafka em KRaft. Esta fase nao implementa regra de negocio.

## Servicos

- `auth-service`: <http://localhost:3001>
- `orders-service`: <http://localhost:3002>
- `inventory-service`: <http://localhost:3003>
- `shipping-service`: <http://localhost:3004>
- `notification-service`: <http://localhost:3005>

## Rodar localmente

```bash
docker compose up --build
```

Kafka roda em modo KRaft e pode levar alguns segundos para registrar o broker. Os microsservicos aguardam o healthcheck do Kafka antes de iniciar.

Se restarem containers antigos de tentativas anteriores:

```bash
docker compose down --remove-orphans
docker compose up --build
```

## Health checks

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

## Publicar eventos demo

```bash
curl -X POST http://localhost:3001/events/demo
curl -X POST http://localhost:3002/events/demo
curl -X POST http://localhost:3003/events/demo
curl -X POST http://localhost:3004/events/demo
curl -X POST http://localhost:3005/events/demo
```

Cada servico publica em seu topico demo e tambem possui um consumidor do mesmo topico com `console.log`.
