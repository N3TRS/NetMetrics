# 📊 NetMetrics — Stack de Observabilidad de OmniCode

<div align="center">

### 🛠️ Stack Tecnológico

![Prometheus](https://img.shields.io/badge/Prometheus-Metrics-E6522C?style=for-the-badge&logo=prometheus&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-Dashboards-F46800?style=for-the-badge&logo=grafana&logoColor=white)
![Loki](https://img.shields.io/badge/Loki-Log_Aggregation-F46800?style=for-the-badge&logo=grafana&logoColor=white)

### ☁️ Infraestructura

![Docker Compose](https://img.shields.io/badge/Docker_Compose-3.8-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Azure](https://img.shields.io/badge/Azure-Targets-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white)

### 🏗️ Arquitectura

![Observability](https://img.shields.io/badge/Pattern-Pull_Model-blueviolet?style=for-the-badge)
![IaC](https://img.shields.io/badge/Config-Infrastructure_as_Code-009688?style=for-the-badge)

</div>

---

## 📑 Tabla de Contenidos

1. [👤 Integrantes](#1--integrantes)
2. [🎯 Objetivo del Proyecto](#2--objetivo-del-proyecto)
3. [⚡ Servicios del Stack](#3--servicios-del-stack)
4. [📋 Estrategia de Versionamiento](#4--estrategia-de-versionamiento)
5. [🧩 Targets de Monitoreo](#5--targets-de-monitoreo)
6. [🏛️ Arquitectura de Observabilidad](#6-️-arquitectura-de-observabilidad)
7. [🗂️ Organización del Código](#7-️-organización-del-código)
8. [🚀 Ejecución del Stack](#8--ejecución-del-stack)
9. [🔍 Verificación y Uso](#9--verificación-y-uso)
10. [🤝 Integrantes y Contribuciones](#10--integrantes-y-contribuciones)

---

## 1. 👤 Integrantes

- Tulio Riaño Sánchez
- Julian Camilo Lopez Barrero
- Juan Sebastián Puentes Julio
- David Alejandro Patacon Henao

---

## 2. 🎯 Objetivo del Proyecto

**NetMetrics** es el stack de observabilidad de la plataforma **OmniCode**. Provee monitoreo en tiempo real de todos los microservicios mediante tres componentes: **Prometheus** para scraping y almacenamiento de métricas, **Grafana** para visualización y dashboards, y **Loki** para agregación de logs. Todo el stack se levanta con un solo comando Docker Compose y monitorea 5 microservicios desplegados en Azure.

---

## 3. ⚡ Servicios del Stack

| Servicio | Puerto (host) | Descripción |
|---|---|---|
| **Prometheus** | 9090 | Recolección y almacenamiento de métricas. Scrape cada 15s. |
| **Grafana** | 4000 | Visualización de métricas y logs. Datasources provisionados automáticamente. |
| **Loki** | 3100 | Agregación y consulta de logs de aplicación. |

---

## 4. 📋 Estrategia de Versionamiento

### Convenciones para commits

```
feat: agregar target omnicode-api-sessions a prometheus.yml
fix: corregir datasource Prometheus en Grafana provisioning
chore: actualizar imagen base de Grafana a latest
docs: agregar sección de verificación en README
```

---

## 5. 🧩 Targets de Monitoreo

### Job `net-services` — Prometheus

Scrape HTTPS cada 15s en `/metrics` de los microservicios Azure:

| Microservicio | Host Azure | Puerto |
|---|---|---|
| **NetAuthentication** | `omnicode-api-authentication.azurewebsites.net` | 443 |
| **NetCalls** | `omnicode-api-calls.azurewebsites.net` | 443 |
| **NetSessions** | `omnicode-api-real-time.azurewebsites.net` | 443 |
| **NetSessions (session)** | `omnicode-api-session.azurewebsites.net` | 443 |
| **NetAI** | `omnicode-api-python.azurewebsites.net` | 443 |

### Métricas disponibles (por microservicio)

| Métrica | Tipo | Descripción |
|---|---|---|
| `http_requests_total` | Counter | Total de peticiones HTTP (labels: method, status, route) |
| `http_request_duration_seconds` | Histogram | Latencia de requests (buckets: 0.1s → 10s) |

### Datasources Grafana (provisionados automáticamente)

| Datasource | URL | Uso |
|---|---|---|
| **Prometheus** | Azure Container Apps endpoint | Métricas de todos los microservicios |
| **Loki** | `http://loki:3100` | Logs agregados |

---

## 6. 🏛️ Arquitectura de Observabilidad

### Modelo Pull (Prometheus)

```
Microservicios Azure                NetMetrics Docker Stack
         │                                  │
omnicode-api-authentication:443/metrics ────►│
omnicode-api-calls:443/metrics ─────────────►│  Prometheus
omnicode-api-real-time:443/metrics ─────────►│  (pull cada 15s)
omnicode-api-session:443/metrics ───────────►│
omnicode-api-python:443/metrics ────────────►│
                                             │
                                        Grafana (4000)
                                             │
                                   Dashboard + Alertas
                                             │
                                        Loki (3100)
                                     Log aggregation
```

### Persistencia de Datos

Los datos de cada servicio se guardan en volúmenes Docker nombrados:

| Volumen | Servicio | Contenido |
|---|---|---|
| `prometheus-data` | Prometheus | Series temporales de métricas |
| `grafana-data` | Grafana | Dashboards, usuarios, configuración |
| `loki-data` | Loki | Streams de logs |

> Los volúmenes persisten con `docker compose down`. Se eliminan solo con `docker compose down -v`.

---

## 7. 🗂️ Organización del Código

```
NetMetrics/
│
├── docker-compose.yml              # Orquestación de los 3 servicios
│
├── prometheus/
│   ├── Dockerfile                  # Imagen base oficial Prometheus
│   └── prometheus.yml              # Scrape config: self + net-services job (5 Azure targets)
│
├── grafana/
│   ├── Dockerfile                  # Imagen base oficial Grafana
│   └── provisioning/
│       └── datasources/
│           └── datasources.yml     # Auto-provisiona Prometheus + Loki datasources
│
├── loki/
│   ├── Dockerfile                  # Imagen base oficial Loki
│   └── loki-config.yml             # Storage local, schema boltdb-shipper
│
└── README.md
```

---

## 8. 🚀 Ejecución del Stack

### 📋 Prerrequisitos

- **Docker >= 20.10** y **Docker Compose >= 2.0**

### 🛠️ Inicio Rápido

```bash
# Clonar el repositorio
git clone <repo-url>
cd NetMetrics

# Levantar el stack completo
docker compose up -d --build
```

### 🌐 URLs de Acceso

| Servicio | URL | Credenciales |
|---|---|---|
| **Grafana** | `http://localhost:4000` | admin / admin |
| **Prometheus** | `http://localhost:9090` | — |
| **Loki** | `http://localhost:3100` | — |

---

## 9. 🔍 Verificación y Uso

### Verificar targets Prometheus

1. Abrir `http://localhost:9090`
2. Ir a **Status → Targets**
3. Verificar que los 5 targets de `net-services` estén en estado `UP`

### Explorar métricas en Grafana

1. Abrir `http://localhost:4000` (admin/admin)
2. Ir a **Explore** y seleccionar datasource `Prometheus`
3. Consultar `http_requests_total` o `http_request_duration_seconds`

### Comandos útiles

```bash
# Levantar o reconstruir
docker compose up -d --build

# Ver logs en tiempo real
docker compose logs -f

# Ver logs de un servicio específico
docker compose logs -f prometheus

# Reiniciar servicios
docker compose restart

# Detener conservando volúmenes
docker compose down

# Detener y eliminar volúmenes (borra todos los datos)
docker compose down -v

# Estado de contenedores
docker compose ps
```

---

## 10. 🤝 Integrantes y Contribuciones

<div align="center">

![Course](https://img.shields.io/badge/Course-ARSW-orange?style=for-the-badge)
![Year](https://img.shields.io/badge/Year-2026--1-blue?style=for-the-badge)

| 👤 Integrante | 🎓 Rol |
|:---|:---|
| Tulio Riaño Sánchez | Desarrollo y arquitectura |
| Julian Camilo Lopez Barrero | Desarrollo y arquitectura |
| Juan Sebastián Puentes Julio | Desarrollo y arquitectura |
| David Alejandro Patacon Henao | Desarrollo y arquitectura |

> 💡 **NetMetrics** centraliza la observabilidad de OmniCode: Prometheus extrae métricas de 5 microservicios Azure, Grafana las visualiza en dashboards, y Loki agrega todos los logs — todo con un solo `docker compose up`.

**🎓 Escuela Colombiana de Ingeniería Julio Garavito**

</div>
