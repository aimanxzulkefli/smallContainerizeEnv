# smallContainerizeEnv

A simple containerized environment showcasing how a Java EE application (deployed on WildFly) interacts with a PostgreSQL database using Docker on Ubuntu.

> This project demonstrates how to build, run, and connect two containers — one for a Java application and one for PostgreSQL.

---

## 📁 Project Structure

![image](https://github.com/user-attachments/assets/e0a45c18-eccf-473a-b86c-bdb2dd66aead)

## Steps to Set Up

### 1. Create a Project Directory

```bash
mkdir /home/myapp/deployments
cd /home/myapp/deployments
```

### 2. Add Your Java Application

Place your compiled Java EE application (VarianceReportEAR.ear) into the deployments folder.

### 3. Create a Dockerfile

Use this Dockerfile to create a custom WildFly image that includes your application:
```bash
vim Dockerfile
```
```
FROM jboss/wildfly

ENV DATABASE_URL=jdbc:postgresql://c-postgres:5432/mydb
ENV DATABASE_USERNAME=${username}
ENV DATABASE_PASSWORD=${pass}

COPY VarianceReportEAR.ear /opt/jboss/wildfly/standalone/deployments/
```

### 4. Build the Wildfly Docker Image

"variance-report-app" is the name given to the image. The "." in the command refers to the location of the Dockerfile

```
docker build -t variance-report-app .
```

### 5. Create docker compose file

Docker Compose is used to define a multi-container application and allows the containers to communicate with each other over a network. In this example, the PostgreSQL and Wildfly containers are defined within the same network, bridge, which is the default network driver in Docker.

The volume (pgdata) ensures that PostgreSQL data persists between container restarts or recreations. Without volumes, data inside the container would be lost when the container is deleted.

The wildfly container connects to the postgres container using the DATABASE_URL (jdbc:postgresql://postgres:5432/mydb) and passes the appropriate credentials via environment variables. The Wildfly application exposes ports 8080 for web services and 8443 for API calls, which are mapped to the host machine's corresponding ports.
```
vim docker-compose.yml
```
```
version: '3.8'

services:
  postgres:
    image: postgres
    container_name: c-postgres
    environment:
      POSTGRES_USER: ${username}
      POSTGRES_PASSWORD: ${pass}
      POSTGRES_DB: mydb
    volumes:
      - pgdata:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    networks:
      - bridge

  wildfly:
    image: variance-report-app
    container_name: c-wildfly
    depends_on:
      - postgres
    ports:
      - "8080:8080"
      - "8443:8443"
    environment:
      DATABASE_URL: jdbc:postgresql://postgres:5432/mydb
      DATABASE_USERNAME: ${username}
      DATABASE_PASSWORD: ${pass}
    networks:
      - bridge

volumes:
  pgdata:

networks:
  bridge:
    driver: bridge
```

### 6. Run both container using docker compose file

```bash
docker-compose up -d
```




