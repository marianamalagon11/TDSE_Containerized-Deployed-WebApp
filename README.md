# TDSE Containerized & Deployed Web App

Mariana Malagón

## Purpose

This project explores virtualization as an architectural mechanism for modularity, isolation, portability, and deployment. It implements a small Java web application built with Spring Boot, packaged as a Docker image, run locally in isolated containers, published to Docker Hub, and deployed on an Amazon EC2 virtual machine.

## Technology stack

- Java 21 LTS
- Maven 3.9+
- Spring Boot 4.1.1
- Docker Desktop with Docker Compose v2
- Docker Hub
- Amazon Linux 2023 on AWS EC2
- Amazon Corretto 21 container image

## Part 1: Web application

A minimal REST web application built with Spring Boot.

**Endpoint:** `GET /greeting?name={name}`, returns `Hello, {name}!` (defaults to `World` if no `name` is provided).

**Environment-based configuration:** the application reads its listening port from the `PORT` environment variable. This is handled in `RestServiceApplication` via `application.setDefaultProperties(...)`, which sets `server.port` before the Spring context starts. The instructor confirmed that the exact default port number is not important, as long as the application correctly reads it from the `PORT` environment variable, so this project was verified using `PORT=7000`.

### Project structure

```
src/main/java/co/edu/escuelaing/virtualizationlab/
├── RestServiceApplication.java   # Application entry point; reads PORT env var
└── HelloRestController.java      # REST controller exposing /greeting
```

### Build and run locally

```bash
mvn clean package
java -jar target/*.jar
```

To run it with the `PORT` environment variable set to `7000`:

```powershell
# Windows PowerShell (wildcards aren't expanded, so use the exact jar name)
$env:PORT=7000
java -jar target/virtualization-lab-1.0.0.jar
```

```bash
# Linux/macOS
PORT=7000 java -jar target/*.jar
```

### Verify

```
http://localhost:7000/greeting?name=Mari
```

Expected response:

```
Hello, Mari!
```

### Evidence

Local execution with the `PORT` environment variable set to `7000`, confirming both the endpoint logic and the environment-based port configuration:

![Local execution evidence](images/01-evidence.png)

This was tested locally: the app reads the `PORT` environment variable (verified with `PORT=7000`), and the `/greeting` endpoint responds correctly with both a provided `name` and the `World` default.

## Part 2: Docker image and containers

The application is packaged into a Docker image based on `amazoncorretto:21` (see [Dockerfile](Dockerfile)).

### Build the image

```bash
mvn clean package
docker build -t marianamalagon11/virtualization-lab:1.0 .
```

![docker build output](images/02-command.png)

Verify the image was created:

```bash
docker images
```

![docker images output](images/02-evidence1.png)

![Docker Desktop Images tab](images/02-evidence2.png)

### Run a container

```bash
docker run -d --name virtualization-lab-1 -e PORT=6000 -p 34000:6000 marianamalagon11/virtualization-lab:1.0
```

![docker run output](images/02-command2.png)

![Docker Desktop Containers tab showing virtualization-lab-1 running](images/02-evidence3.png)

The container maps host port `34000` to the container's internal port `6000`, and its logs confirm Spring Boot started successfully.

![Browser test of virtualization-lab-1 on port 34000](images/02-evidence6.png)

### Demonstrating container isolation

Two additional containers were started from the same image on different host ports. They run independently of each other and of `virtualization-lab-1`, proving that Docker gives each instance its own isolated process and network space on the same host:

```bash
docker run -d --name virtualization-lab-2 -p 34001:6000 marianamalagon11/virtualization-lab:1.0
docker run -d --name virtualization-lab-3 -p 34002:6000 marianamalagon11/virtualization-lab:1.0
```

![docker run output for containers 2 and 3](images/02-command2.png)

![Docker Desktop Containers list showing all three running with their port mappings](images/02-evidence5.png)

Each container was verified independently and responded with its own name parameter, confirming they are fully isolated instances:

```
http://localhost:34000/greeting?name=Container   -> Hello, Container!
http://localhost:34001/greeting?name=Container2  -> Hello, Container2!
http://localhost:34002/greeting?name=Container3  -> Hello, Container3!
```

![Side-by-side browser evidence: container 2 and container 3 responding independently](images/02-evidence4.png)
