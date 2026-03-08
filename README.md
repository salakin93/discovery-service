# Discovery Service

El **Discovery Service** es el servidor de **Service Discovery** de la plataforma de microservicios.
Está basado en **Netflix Eureka** y permite que todos los microservicios del sistema se registren y puedan descubrirse dinámicamente entre sí.

En una arquitectura de microservicios, los servicios no deberían depender de **IP o puertos fijos**. En su lugar, utilizan un registro central donde anuncian su disponibilidad.

Este servicio cumple esa función.

---

# Responsabilidad del servicio

El Discovery Service actúa como **registro central de microservicios**.

Permite:

* registrar microservicios automáticamente
* descubrir servicios disponibles
* balancear tráfico entre instancias
* evitar configuraciones estáticas de red

---

# Arquitectura

El Discovery Service forma parte de la infraestructura central del sistema.

```text
Client
   │
   ▼
API Gateway
   │
   ▼
Service Discovery (Eureka)
   │
   ▼
Microservices
```

Flujo de funcionamiento:

1. Un microservicio inicia.
2. El microservicio se registra en **Eureka Server**.
3. Otros servicios consultan Eureka para encontrar su ubicación.
4. La comunicación entre servicios se realiza usando el **nombre del servicio**.

---

# Tecnologías utilizadas

| Tecnología                  | Propósito         |
| --------------------------- | ----------------- |
| Spring Boot                 | Framework base    |
| Spring Cloud Netflix Eureka | Service Discovery |
| Spring Boot Actuator        | Monitoreo         |
| Gradle                      | Build tool        |
| Docker                      | Contenerización   |

---

# Estructura del proyecto

```text
discovery-service
│
├── src
│   └── main
│       ├── java
│       │   └── edu.usip.discovery
│       │        └── DiscoveryServiceApplication.java
│       │
│       └── resources
│            └── application.properties
│
├── build.gradle
├── Dockerfile
└── README.md
```

---

# Habilitar Eureka Server

La aplicación habilita el servidor de descubrimiento mediante la anotación:

```
@EnableEurekaServer
```

Ejemplo:

```java
package edu.usip.discovery;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class DiscoveryServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(DiscoveryServiceApplication.class, args);
    }

}
```

---

# Configuración

Este servicio obtiene su configuración desde **Config Server**.

Archivo:

```text
src/main/resources/application.properties
```

```properties
spring.application.name=discovery-service
spring.config.import=configserver:http://config-service:8888
```

---

# Configuración en microservices-config

La configuración principal del servicio se encuentra en el repositorio:

```
microservices-config
```

Archivo:

```
discovery-service.properties
```

```properties
server.port=8761
spring.application.name=discovery-service

eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false

management.endpoints.web.exposure.include=health,info
```

### Explicación

| Propiedad            | Descripción                                          |
| -------------------- | ---------------------------------------------------- |
| server.port          | Puerto del servidor Eureka                           |
| register-with-eureka | El servidor no debe registrarse a sí mismo           |
| fetch-registry       | El servidor no obtiene registros de otros servidores |
| actuator             | Endpoints de monitoreo                               |

---

# Dashboard de Eureka

Una vez iniciado el servicio se puede acceder al panel web en:

```
http://localhost:8761
```

Desde este panel se pueden visualizar:

* microservicios registrados
* estado de las instancias
* información del servidor Eureka

---

# Integración con otros microservicios

Cada microservicio que quiera registrarse en Eureka debe incluir:

```properties
eureka.client.service-url.defaultZone=http://discovery-service:8761/eureka
```

y tener configurado:

```properties
spring.application.name=<nombre-del-servicio>
```

Ejemplo para `identity-service`:

```properties
spring.application.name=identity-service
eureka.client.service-url.defaultZone=http://discovery-service:8761/eureka
```

---

# Dockerización

El servicio está preparado para ejecutarse dentro de un contenedor Docker.

---

# Dockerfile

```dockerfile
FROM gradle:8.7-jdk21 AS builder

WORKDIR /app
COPY . .

RUN gradle bootJar --no-daemon

FROM eclipse-temurin:21-jdk

WORKDIR /app
COPY --from=builder /app/build/libs/*.jar app.jar

EXPOSE 8761

ENTRYPOINT ["java","-jar","/app/app.jar"]
```

---

# Ejecución con Docker

Construir la imagen:

```
docker build -t discovery-service .
```

Ejecutar el contenedor:

```
docker run -p 8761:8761 discovery-service
```

---

# Integración con Docker Compose

Este servicio se ejecutará junto con el resto de la infraestructura usando **Docker Compose**.

Ejemplo simplificado:

```yaml
services:

  discovery-service:
    build: ./discovery-service
    container_name: discovery-service
    ports:
      - "8761:8761"
```

Los microservicios podrán acceder al servidor Eureka utilizando:

```
http://discovery-service:8761
```

Docker Compose crea automáticamente una red interna entre contenedores.

---

# Próximos servicios en la arquitectura

Después de este servicio se implementará:

* gateway-service (API Gateway)
* document-service
* ai-service
* notification-service
* telegram-bot-service

Todos estos servicios se registrarán automáticamente en **Discovery Service**.

---

# Licencia

Proyecto académico / investigación.
