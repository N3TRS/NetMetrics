# NetMetrics

Stack de observabilidad para tus microservicios: **Prometheus**, **Grafana** y **Loki**.

## Servicios

| Servicio | Puerto | Descripción |
|----------|--------|-------------|
| Grafana | 4000 | Visualización de métricas y dashboards |
| Prometheus | 9090 | Recolección y almacenamiento de métricas |
| Loki | 3100 | Agregación de logs |

## Quick Start

```bash
docker-compose up -d
```

## Acceder a los servicios

- **Grafana**: http://localhost:4000 (admin/admin)
- **Prometheus**: http://localhost:9090
- **Loki**: http://localhost:3100

## Estructura

```
NetMetrics/
├── docker-compose.yml              # Orquestación del stack
├── prometheus/
│   └── prometheus.yml              # Config de scraping
├── grafana/
│   └── provisioning/
│       ├── datasources/            # Data sources auto-config
│       └── dashboards/              # Dashboards provisionados
├── loki/
│   └── loki-config.yml             # Config de Loki
└── promtail/
    └── promtail-config.yml         # Recolector de logs
```

## Microservicios monitoreados

| Microservicio | Puerto | Puerto Docker |
|---------------|--------|---------------|
| NetAuthentication | 3000 | host.docker.internal:3000 |
| NetSession | 3002 | host.docker.internal:3002 |
| NetCalls | 3003 | host.docker.internal:3003 |
| NetRealTimeEditor | 3004 | host.docker.internal:3004 |

## Configurar tus microservicios

### 1. Instalar dependencias

```bash
npm install prom-client
```

### 2. Crear `src/metrics.controller.ts`

```typescript
import { Controller, Get } from '@nestjs/common';
import { Registry, Counter, Histogram } from 'prom-client';

@Controller('metrics')
export class MetricsController {
  private registry: Registry;
  private httpRequestsTotal: Counter;
  private httpRequestDuration: Histogram;

  constructor() {
    this.registry = new Registry();

    this.httpRequestsTotal = new Counter({
      name: 'http_requests_total',
      help: 'Total number of HTTP requests',
      labelNames: ['method', 'status', 'route'],
      registers: [this.registry],
    });

    this.httpRequestDuration = new Histogram({
      name: 'http_request_duration_seconds',
      help: 'Duration of HTTP requests in seconds',
      labelNames: ['method', 'route'],
      buckets: [0.1, 0.5, 1, 2, 5, 10],
      registers: [this.registry],
    });
  }

  @Get()
  async getMetrics() {
    return this.registry.metrics();
  }
}
```

### 3. Agregar controller en `app.module.ts`

```typescript
import { MetricsController } from './metrics.controller';

@Module({
  controllers: [MetricsController],
  // ...
})
export class AppModule {}
```

## Verificar que funciona

1. Inicia el stack: `docker-compose up -d`
2. Ve a Prometheus: http://localhost:9090
3. Status → Targets
4. Deberías ver tus 4 microservicios en estado "UP"

## Queries PromQL útiles

| Métrica | Query |
|---------|-------|
| Requests por segundo | `sum(rate(http_requests_total[1m]))` |
| Latencia promedio | `sum(rate(http_request_duration_seconds_sum[1m])) / sum(rate(http_request_duration_seconds_count[1m]))` |
| Latencia p99 | `histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))` |
| Requests por ruta | `sum by (route) (http_requests_total)` |
| Errors 5xx | `sum(rate(http_requests_total{status=~"5.."}[1m]))` |

## Comandos útiles

```bash
# Iniciar stack
docker-compose up -d

# Ver logs
docker-compose logs -f

# Reiniciar servicios
docker-compose restart

# Detener (mantiene datos)
docker-compose down

# Detener y eliminar todo
docker-compose down -v
```

## Persistencia

Los dashboards de Grafana se guardan en el volumen `grafana-data` y sobreviven a `docker-compose down`. Se pierden solo con `docker-compose down -v`.
