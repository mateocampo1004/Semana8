
# Despliegue Automatizado de un Backend con PostgreSQL y pgAdmin usando Docker Compose

## 1. Título  
**Despliegue Automatizado de un Backend con PostgreSQL y pgAdmin usando Docker Compose**

## 2. Tiempo de duración  
**40 minutos**

## 3. Fundamentos  

En esta práctica se implementó un entorno de desarrollo backend utilizando Docker y Docker Compose. El objetivo fue automatizar el despliegue de una aplicación backend basada en Spring Boot y Kotlin, conectada a una base de datos PostgreSQL, con administración visual a través de pgAdmin. Para ello, se utilizó un `docker-compose.yml` con configuración multi-servicio y un `Dockerfile` con estrategia de multi-stage build.

### Comandos utilizados  

- `docker compose up -d`: Levantar todos los contenedores en segundo plano.  
- `docker compose down`: Detener y eliminar los contenedores.  
- `docker compose logs -f backend`: Ver los logs en tiempo real del backend.  
- `docker compose build backend`: Construir manualmente la imagen del backend.

## 4. Conocimientos previos

- Fundamentos de Docker y Docker Compose.  
- Conocimiento básico de backend en Spring Boot o equivalente.  
- Manejo de bases de datos relacionales como PostgreSQL.  
- Familiaridad con puertos, variables de entorno y volúmenes en Docker.

## 5. Objetivos a alcanzar

- Configurar un entorno multi-contenedor con Docker Compose.  
- Desplegar un backend funcional que se conecte a PostgreSQL.  
- Visualizar y administrar la base de datos con pgAdmin.  
- Implementar una imagen optimizada usando multi-stage build.  
- Comprobar la ejecución de endpoints reales desde Postman.

## 6. Equipo necesario

- PC con Docker Desktop y WSL (Ubuntu)  
- Editor de texto (Visual Studio Code, nano, etc.)  
- Navegador web  
- Postman o curl para pruebas de endpoints

## 7. Material de apoyo

- [Docker Compose Docs](https://docs.docker.com/compose/)  
- [Spring Boot](https://spring.io/projects/spring-boot)  
- [PostgreSQL](https://www.postgresql.org/docs/)

## 8. Procedimiento

### Paso 1: Clonar el proyecto base

```bash
mkdir backend-docker
cd backend-docker
git clone https://github.com/maguaman2/tendencias-mar22-security.git backend
```

### Paso 2: Crear archivo `.env`

```env
POSTGRES_USER=admin
POSTGRES_PASSWORD=admin123
POSTGRES_DB=appdb
PGADMIN_EMAIL=admin@admin.com
PGADMIN_PASSWORD=admin123
```

### Paso 3: Crear archivo `docker-compose.yml`

```yaml
services:
  db:
    image: postgres:15
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    volumes:
      - db-data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    networks:
      - app-net

  pgadmin:
    image: dpage/pgadmin4
    environment:
      PGADMIN_DEFAULT_EMAIL: ${PGADMIN_EMAIL}
      PGADMIN_DEFAULT_PASSWORD: ${PGADMIN_PASSWORD}
    ports:
      - "8083:80"
    depends_on:
      - db
    networks:
      - app-net

  backend:
    build: ./backend
    ports:
      - "8081:8081"
    depends_on:
      - db
    environment:
      DB_SERVER: db
      DB_PORT: 5432
      DB_NAME: appdb
      DB_USER: admin
      DB_PASSWORD: admin123
    networks:
      - app-net

volumes:
  db-data:

networks:
  app-net:
```

### Paso 4: Crear el `Dockerfile` con multi-stage build

```dockerfile
FROM eclipse-temurin:21-jdk as build
RUN apt-get update && apt-get install -y maven
WORKDIR /app
COPY . .
RUN mvn clean package -DskipTests

FROM eclipse-temurin:21-jdk
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Paso 5: Levantar y probar los servicios

```bash
docker compose up -d --build
```

- Accede a **pgAdmin** en: http://localhost:8083  
- Accede al backend en: http://localhost:8081/users

## 9. Resultados

Se logró desplegar un backend funcional dentro de un contenedor Docker, conectado correctamente a PostgreSQL y accesible mediante red local. Flyway aplicó la migración SQL correctamente, y la base fue visible desde pgAdmin. El backend respondió exitosamente al endpoint `/users`, evidenciando una integración total entre servicios y correcta configuración de variables de entorno.

## 10. Conclusiones

Esta práctica demostró cómo un entorno backend completo puede ser automatizado usando Docker Compose, incluyendo base de datos, panel de administración y aplicación compilada con multi-stage build. La reutilización del proyecto base permitió enfocarse en la integración y el despliegue, mientras que Docker Compose facilitó la orquestación y persistencia de datos.

## 11. Bibliografía

- Docker. (2024). *Docker Docs*. https://docs.docker.com/  
- Spring Boot. (2024). *Spring Documentation*. https://spring.io/projects/spring-boot  
- PostgreSQL. (2024). *PostgreSQL Documentation*. https://www.postgresql.org/docs/
