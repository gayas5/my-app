
---

# 🚀 Project Upgrade: `myapp`

We will implement **ALL** of these:

1. Convert Java app → **Web App**
2. Add **Dockerfile**
3. **CI/CD using GitHub Actions**
4. **Jenkins Pipeline**
5. **Kubernetes Deployment**

---

# 1️⃣ Convert Java App → Web App (Spring Boot)

## Why Spring Boot?

* Embedded Tomcat (no external server needed)
* Industry standard
* Easy Docker & Kubernetes support

---

## 📁 Updated Folder Structure

```
myapp/
├── my-app/
│   ├── pom.xml
│   ├── Dockerfile
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/com/example/
│   │   │   │   └── MyAppApplication.java
│   │   │   │   └── controller/HelloController.java
│   │   │   └── resources/
│   │   │       └── application.properties
│   │   └── test/
│   └── target/
├── k8s/
│   ├── deployment.yaml
│   └── service.yaml
├── Jenkinsfile
├── .github/
│   └── workflows/
│       └── ci-cd.yml
└── README.md
```

---

## 🛠 Step 1: Update `pom.xml` (Spring Boot Web)

Replace existing `pom.xml` with:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>my-app</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <name>My Spring Boot Web App</name>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.4</version>
    </parent>

    <properties>
        <java.version>21</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

---

## 🧠 Step 2: Spring Boot Main Class

`src/main/java/com/example/MyAppApplication.java`

```java
package com.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MyAppApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyAppApplication.class, args);
    }
}
```

---

## 🌐 Step 3: REST Controller

`src/main/java/com/example/controller/HelloController.java`

```java
package com.example.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/")
    public String hello() {
        return "Hello from Spring Boot Web App!";
    }
}
```

---

## ▶ Build & Run (Docker Maven)

```bash
docker run -it --rm \
  -v "$(pwd)":/usr/src/mymaven \
  -w /usr/src/mymaven/my-app \
  maven:3.9.12-eclipse-temurin-21-alpine \
  mvn clean package
```

Run locally:

```bash
java -jar target/my-app-1.0.0.jar
```

Visit:

```
http://localhost:8080
```

---

# 2️⃣ Add Dockerfile

## 📄 `my-app/Dockerfile`

```dockerfile
FROM eclipse-temurin:21-jre-alpine

WORKDIR /app

COPY target/my-app-1.0.0.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java","-jar","app.jar"]
```

---

## 🐳 Build & Run Docker Image

```bash
docker build -t myapp:1.0 .
docker run -d -p 8080:8080 myapp:1.0
```

---

# 3️⃣ CI/CD Using GitHub Actions

## 📁 `.github/workflows/ci-cd.yml`

```yaml
name: CI-CD Pipeline

on:
  push:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Set up Java
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 21

      - name: Build with Maven
        run: mvn clean package
        working-directory: my-app

      - name: Build Docker Image
        run: docker build -t myapp:${{ github.sha }} my-app
```

### ✅ What This Does

* Triggers on `git push`
* Builds JAR
* Builds Docker image
* Fully automated CI

---

# 4️⃣ Jenkins Pipeline

## 📄 `Jenkinsfile`

```groovy
pipeline {
    agent any

    tools {
        maven 'maven-3'
        jdk 'java-21'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/<your-username>/myapp.git'
            }
        }

        stage('Build') {
            steps {
                dir('my-app') {
                    sh 'mvn clean package'
                }
            }
        }

        stage('Docker Build') {
            steps {
                dir('my-app') {
                    sh 'docker build -t myapp:latest .'
                }
            }
        }
    }
}
```

### Jenkins Requirements

* Maven configured
* JDK 21 configured
* Docker installed on Jenkins node

---

# 5️⃣ Kubernetes Deployment

## 📁 `k8s/deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:latest
        ports:
        - containerPort: 8080
```

---

## 📁 `k8s/service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30007
```

---

## 🚀 Deploy to Kubernetes (Minikube)

```bash
minikube start
eval $(minikube docker-env)

docker build -t myapp:latest my-app

kubectl apply -f k8s/
```

Access app:

```bash
minikube service myapp-service
```

---

# ✅ FINAL PIPELINE FLOW (Interview Gold)

```
Git Push
  ↓
GitHub Actions (CI)
  ↓
Docker Image
  ↓
Jenkins (Optional CD)
  ↓
Kubernetes Deployment
```

---

## 🔥 You Now Have

✔ Spring Boot Web App
✔ Dockerized Application
✔ GitHub Actions CI
✔ Jenkins Pipeline
✔ Kubernetes Deployment

---


