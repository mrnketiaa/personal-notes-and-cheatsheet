#### HOW TO CONTAINERIZE REACT, SPRING BOOT AND POSTGRESQL INSIDE DOCKER 

###### This documentation describes the steps that are required to set up a fullstack Spring Boot, React and Postgresql da inside docker.

1. Create your Spring Boot Application
   - Language : Java
   - Type : Maven
   - JDK : 21
   - Java : 21
   - Packaging : Jar
   - Configuration : Yaml
   - Spring Boot : 3.5.14
   - Dependencies
       - Spring Web
       - PostgreSQL Driver
       - Spring Data JPA

[//]: # (The react.js frontend was created inside the same spring boot root directory)

2. Run the docker commands below to setup the react project.
```bash
   # Windows
   docker run --rm -it -v ${PWD} :/code -w /code node:latest sh -c "npm create vite@latest frontend -- --template react && cd frontend / && npm install"          
```

3. Create `.dockerignore` file and include the following base content
   - frontend/node_modules
   - frontend/dist
   - frontend/.git
   - .dockerignore

4. Edit the `vite.config.js` file by adding the following lines
```javascript
   server:{host: "0.0.0.0"},
   build: {outDir: "dist"}
```
   
4. Create the `Dockerfile` and include the following base content
```bash
FROM node:20-alpine AS react-builder

COPY ./frontend /frontend
WORKDIR /frontend
RUN npm ci
RUN npm run build 


FROM maven:3.9.9-eclipse-temurin-21 AS builder

WORKDIR /app

COPY pom.xml .
COPY src ./src

COPY --from=react-builder /frontend/dist ./src/main/resources/static
RUN mvn clean package -DskipTests


FROM eclipse-temurin:21-jre AS runner

WORKDIR /app
COPY --from=builder /app/target/react-spring-0.0.1-SNAPSHOT.jar app.jar

EXPOSE 4002

ENTRYPOINT ["java", "-jar", "app.jar"]
```

5. Create the `docker-compose.yml` file and include the following base content 
```bash
services:
  frontend:
    build:
      context: .
    ports:
      - "4002:8080"   # host:container


    # postgresql set-up
    environment:
      - SPRING_DATASOURCE_URL=jdbc:postgresql://db:5432/reactspring
      - SPRING_DATASOURCE_USERNAME=postgres
      - SPRING_DATASOURCE_PASSWORD=postgres

    depends_on:
      db:
        condition: service_healthy

    # automatically rebuild the project without manually restarting
    develop:
      watch:
        - action: sync
          path: ./frontend
          target: ./frontend
          ignore:
            - node_modules

        - action: rebuild
          path: frontend/src
          ignore:
            - node_modules
        - action: rebuild
          path: ./src/main/java

  db:
    image: postgres:16-alpine
    environment:
      - POSTGRES_DB=reactspring
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
```

[//]: # (OPTIONAL: Docker command to install react.js dependency)
```bash
   docker compose run --run --v ${PWD}/frontend: /frontend sh -c "npm install axios"
```

6. Run the docker commands below to build the application
```bash
   docker compose up --watch --build
```

7. Configuring `CORS` to receive data from the react frontend
#### Option 1 : Without Spring Security 
- Using a Configuration class `(corsConfig.java)`

```java
    import org.springframework.web.servlet.config.annotation.CorsRegistry;
    import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
    
    @Configuration
    public class corsConfig implements WebMvcConfigurer {
    
        @Override
        public void addCorsMappings(CorsRegistry registry){
            WebMvcConfigurer.super.addCorsMappings(registry);
            registry.addMapping("/api/endpoint").allowedOrigins("http://localhost:[frontend-port]")
        }
    }
```

#### Optional 2 : Using Spring Security 
[included in the Spring Security Section]
