# NetMetrics

Stack de observabilidad con Docker Compose para Prometheus, Grafana y Loki.

## Servicios

| Servicio | Puerto host | Descripcion |
|----------|-------------|-------------|
| Grafana | 4000 | Visualizacion y consultas |
| Prometheus | 9090 | Recoleccion y almacenamiento de metricas |
| Loki | 3100 | Almacenamiento de logs |

## Inicio rapido

```bash
docker compose up -d --build
```

## Acceso

- Grafana: http://localhost:4000 (admin/admin)
- Prometheus: http://localhost:9090
- Loki: http://localhost:3100

## Estructura real del proyecto

```text
NetMetrics/
|- docker-compose.yml
|- README.md
|- grafana/
|  |- Dockerfile
|  |- provisioning/
|     |- datasources/
|        |- datasources.yml
|- loki/
|  |- Dockerfile
|  |- loki-config.yml
|- prometheus/
   |- Dockerfile
   |- prometheus.yml
```

## Configuracion actual

- Prometheus scrapea:
  - `localhost:9090` (self-scrape)
  - Endpoints HTTPS de Azure Web Apps en `/metrics`
- Grafana tiene dos datasources provisionados:
  - `Prometheus` apuntando a un endpoint remoto (Azure Container Apps)
  - `Loki` apuntando al contenedor local `http://loki:3100`

## Targets actuales en Prometheus

Los targets definidos para el job `net-services` son:

- `omnicode-api-authentication.azurewebsites.net:443`
- `omnicode-api-calls.azurewebsites.net:443`
- `omnicode-api-real-time.azurewebsites.net:443`
- `omnicode-api-session.azurewebsites.net:443`
- `omnicode-api-python.azurewebsites.net:443`

## Verificacion rapida

1. Levanta el stack con `docker compose up -d --build`.
2. Abre Prometheus en http://localhost:9090.
3. Ve a `Status -> Targets` y valida que los endpoints esten `UP`.
4. Abre Grafana en http://localhost:4000 e inicia sesion con `admin/admin`.

## Comandos utiles

```bash
# Levantar o reconstruir
docker compose up -d --build

# Ver logs
docker compose logs -f

# Reiniciar servicios
docker compose restart

# Detener conservando volumenes
docker compose down

# Detener y eliminar volumenes
docker compose down -v
```

## Persistencia

Los datos de Prometheus, Grafana y Loki se guardan en volumenes Docker:

- `prometheus-data`
- `grafana-data`
- `loki-data`

Se eliminan solo con `docker compose down -v`.
