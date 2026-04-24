# Análisis de Implementación de Métricas Seleccionadas

## 1. Arquitectura Propuesta

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│                 │    │                 │    │                 │
│  Aplicación     │───▶│  Middleware     │───▶│  MongoDB        │
│  .NET Core      │    │  de Métricas    │    │  (Métricas)    │
│                 │    │                 │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
        │                       │                       │
        │                       ▼                       │
        │               ┌─────────────────┐            │
        └──────────────▶│  Dashboard de   │◀───────────┘
                        │  Monitoreo      │
                        │  (Grafana)      │
                        └─────────────────┘
```

## 2. Clasificación de Métricas

### 2.1 Métricas de Rendimiento del Sistema
- **Tiempo de respuesta**
- **Uso de CPU**
- **Uso de memoria**
- **Uso de red** (bytes enviados/recibidos)
- **Tasa de errores**
- **Tiempo de base de datos**
- **Uso de conexiones a la base de datos**
- **Operaciones de escritura/lectura por segundo**

### 2.2 Métricas de Seguridad
- **Intentos fallidos de inicio de sesión**
- **Uso de tokens** (creación/renovación)
- **Tiempo de sesión promedio**
- **Tasa de solicitudes sospechosas**
- **Actividad inusual por IP/usuario**

### 2.3 Métricas de Base de Datos
- **Consultas lentas** (>100ms)
- **Índices no utilizados**
- **Tamaño de colecciones/tablas**
- **Uso de índices vs. escaneos completos**

### 2.4 Métricas de Negocio
- **Usuarios activos** - Número de usuarios únicos
- **Transacciones** - Conteo por tipo de operación
- **Disponibilidad**
- **Errores por tipo** (4xx, 5xx)
- **Errores de validación**
- **Errores de tiempo de espera**
- **Errores de concurrencia**

## 3. Tabla de Requisitos por Métrica

| Categoría | Métrica | Requisitos Técnicos | Dependencias Externas | Complejidad |
|-----------|---------|---------------------|------------------------|-------------|
| **Rendimiento** | Tiempo de respuesta | Middleware personalizado con temporizadores | System.Diagnostics.Stopwatch | Baja |
|  | Uso de CPU | PerformanceCounter de .NET | System.Diagnostics | Media |
|  | Uso de memoria | GC y Process de .NET | System.Diagnostics | Baja |
|  | Uso de red | NetworkInterface de .NET | System.Net.NetworkInformation | Media |
|  | Tasa de errores | Filtros de excepciones globales | - | Baja |
|  | Tiempo de base de datos | Interceptores de EF Core | Microsoft.EntityFrameworkCore | Media |
|  | Uso de conexiones | SqlConnectionStringBuilder y estadísticas | Microsoft.Data.SqlClient | Media |
| **Seguridad** | Intentos fallidos | Contador en base de datos | - | Baja |
|  | Uso de tokens | Registro en base de datos con fechas | - | Baja |
|  | Tiempo de sesión | Registro de timestamps | - | Baja |
|  | Actividad sospechosa | Análisis de patrones básico | - | Media |
| **Base de Datos** | Consultas lentas | Monitoreo de consultas SQL Server, registro en MongoDB | Microsoft.Data.SqlClient, MongoDB.Driver | Media |
|  | Índices no utilizados | Análisis de uso de índices en SQL Server, almacenamiento en MongoDB | Microsoft.Data.SqlClient, MongoDB.Driver | Alta |
|  | Tamaño de tablas | Consulta de metadatos de SQL Server, almacenamiento en MongoDB | Microsoft.Data.SqlClient, MongoDB.Driver | Media |
|  | Uso de índices | Análisis de planes de ejecución de SQL Server, almacenamiento en MongoDB | Microsoft.Data.SqlClient, MongoDB.Driver | Alta |
|  | Bloqueos y deadlocks | Monitoreo de bloqueos en SQL Server, registro en MongoDB | Microsoft.Data.SqlClient, MongoDB.Driver | Media |
|  | Estadísticas de conexiones | Monitoreo del pool de conexiones de SQL Server, almacenamiento en MongoDB | Microsoft.Data.SqlClient, MongoDB.Driver | Media |
| **Negocio** | Usuarios activos | Seguimiento de sesiones, registro de actividad | - | Media |
|  | Transacciones | Puntos de instrumentación en lógica de negocio | - | Media |
|  | Disponibilidad | Monitoreo de endpoints, chequeos de salud | Microsoft.AspNetCore.Diagnostics.HealthChecks | Baja |
|  | Errores por tipo | Clasificación centralizada de excepciones | - | Media |

### 3.1 Requisitos Comunes
- **Middleware de Captura**: Para métricas HTTP/API
- **Sistema de Logging Estructurado**: Para seguimiento de eventos
- **Base de Datos de Series de Tiempo**: Para almacenamiento eficiente
- **Sistema de Alertas**: Para notificaciones proactivas
- **Dashboard**: Para visualización de métricas