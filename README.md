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

---

## 🔐 Matriz de Variables de Entorno Requeridas

Para dar cumplimiento al principio de los **Doce Factores (III. Configuraciones en el Entorno)** y garantizar que no existan credenciales hardcodeadas:

| Variable | Propósito | Valor por Defecto (`dev`) | Obligatorio en `prod` |
| :--- | :--- | :--- | :---: |
| `DB_HOST` | Host o nombre del servicio Docker MySQL | `localhost` | **Sí** (ej: `mysql-shipment`) |
| `DB_PORT` | Puerto de escucha de MySQL | `3306` | No (default `3306`) |
| `DB_USER` | Usuario de conexión a la base de datos | `root` | **Sí** |
| `DB_PASSWORD` | Contraseña de conexión a MySQL | `root` | **Sí** (sin fallback) |
| `COURIER_SERVICE_URL` | URL base del microservicio de repartidores | `http://localhost:8081` | `http://courier-service:8081` |

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
