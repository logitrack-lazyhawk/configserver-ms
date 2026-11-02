# LogiTrack - Centralized Configuration Repository (`logitrack-config`)

Repositorio externo de configuración centralizada para el ecosistema de microservicios **LogiTrack**, correspondiente al **Sprint 2** de la cátedra *Desarrollo y Arquitecturas Avanzadas de Software* (DAAS).

Consumido por el microservicio **`config-server`** (Spring Cloud Config Server, puerto `8085`).

---

## 📂 Contenido del Repositorio

El repositorio almacena exclusivamente archivos de configuración estructurados en formato YAML (`2 espacios de sangría`), segregados por microservicio y ambiente:

| Archivo | Microservicio | Perfil | Puerto | Base de Datos |
| :--- | :--- | :--- | :--- | :--- |
| `shipment-service-dev.yml` | `shipment-service` | `dev` | 8080 | `shipment_db` |
| `shipment-service-prod.yml` | `shipment-service` | `prod` | 8080 | `shipment_db` |
| `courier-service-dev.yml` | `courier-service` | `dev` | 8081 | `courier_db` |
| `courier-service-prod.yml` | `courier-service` | `prod` | 8081 | `courier_db` |
| `security-service-dev.yml` | `security-service` | `dev` | 8083 | `security_db` |
| `security-service-prod.yml` | `security-service` | `prod` | 8083 | `security_db` |

---

## 🔐 Matriz de Variables de Entorno Requeridas

Para dar cumplimiento al principio de los **Doce Factores (III. Configuraciones en el Entorno)** y garantizar que no existan credenciales hardcodeadas:

| Variable | Propósito | Valor por Defecto (`dev`) | Obligatorio en `prod` |
| :--- | :--- | :--- | :---: |
| `DB_HOST` | Host o nombre del servicio Docker MySQL | `localhost` | **Sí** (ej: `mysql-shipment`) |
| `DB_PORT` | Puerto de escucha de MySQL | `3306` | No (default `3306`) |
| `DB_NAME` | Nombre de la base de datos | `shipment_db` / `courier_db` / `security_db` | No (usa nombre por defecto) |
| `DB_USER` | Usuario de conexión a la base de datos | `root` | **Sí** |
| `DB_PASSWORD` | Contraseña de conexión a MySQL | `root` | **Sí** (sin fallback) |
| `EUREKA_SERVER_URL` | Registro y descubrimiento para Feign | `http://localhost:8065/eureka/` | `http://eureka-server:8065/eureka/` |
| `ZIPKIN_ENDPOINT` | Exportación de trazas | `http://localhost:9411/api/v2/spans` | `http://zipkin-server:9411/api/v2/spans` |
| `SECURITY_JWT_SECRET_KEY` | Clave secreta privada para firma y validación de tokens JWT | *(hash por defecto dev)* | **Sí** (alta entropía en prod) |
| `SECURITY_JWT_EXPIRATION` | Tiempo de expiración del token JWT en milisegundos | `3600000` (1h) | No |
| `SECURITY_JWT_ISSUER` | Emisor identificado en el token JWT | `logitrack-security` | No |

---

## 🔍 Guía de Verificación con Spring Cloud Config Server

Una vez levantado el servidor **`config-server`** (escuchando en `http://localhost:8085`):

### 1. Verificar estado del Config Server
```bash
curl -X GET http://localhost:8085/actuator/health
```

### 2. Consultar configuración de `shipment-service`
```bash
# Ambiente Dev
curl -X GET http://localhost:8085/shipment-service/dev

# Ambiente Prod
curl -X GET http://localhost:8085/shipment-service/prod

# Formato YAML directo
curl -X GET http://localhost:8085/shipment-service-dev.yml
```

### 3. Consultar configuración de `courier-service`
```bash
curl -X GET http://localhost:8085/courier-service/dev
curl -X GET http://localhost:8085/courier-service/prod
```

### 4. Consultar configuración de `security-service`
```bash
curl -X GET http://localhost:8085/security-service/dev
curl -X GET http://localhost:8085/security-service/prod
```

---

## 🚀 Conexión de los Microservicios Clientes

En cada microservicio cliente (Spring Boot), configure el archivo `src/main/resources/application.yml` (o `bootstrap.yml`) con la importación remota:

```yaml
spring:
  application:
    name: shipment-service # o courier-service / security-service
  config:
    import: "optional:configserver:${CONFIG_SERVER_URL:http://localhost:8085}"
  profiles:
    active: ${SPRING_PROFILES_ACTIVE:dev}
```

## Sprint 3: Feign y trazabilidad

Las propiedades de OpenFeign y Resilience4j están centralizadas en los dos perfiles
de shipment-service. Los errores de negocio (400/404) no cuentan como fallos del
circuito. Las llamadas tienen 2 segundos para conectar, 3 para leer y un límite
global de 6 segundos. El circuito se abre con al menos 5 llamadas y un 50% de fallos,
y espera 10 segundos antes de probar la recuperación.

Spring Boot 4 utiliza Micrometer Tracing + Brave y el starter de Zipkin.
La propiedad del exportador es `management.tracing.export.zipkin.endpoint`.
`TRACING_SAMPLING_PROBABILITY` vale 1.0 para la demostración; puede reducirse por entorno.
La configuración de Eureka está en `eureka-server-dev.yml` y `eureka-server-prod.yml`.
Docker Compose monta este repositorio en modo native; fuera de Docker, Config Server
mantiene su backend Git y la rama main.
