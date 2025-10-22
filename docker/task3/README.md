# Task 3 – Multi-Stage Build for a Java Spring Boot App

## Objective

Build and run a **Spring Boot Java application** inside a container using a **multi-stage Docker build** to reduce image size and separate build and runtime environments.

---

## Application Code

Clone the repository containing the app source:

```bash
git clone https://github.com/Ibrahim-Adel15/Docker-1.git
```

---

## Dockerfile

Create a file named **`Dockerfile`** in your project directory with the following content:

```dockerfile
# -------- Stage 1: Build --------
FROM maven:3.9.4-sapmachine-17 AS build

# Set working directory inside build stage
WORKDIR /app

# Copy project files into the container
COPY . .

# Build the JAR file (skip tests to speed up)
RUN mvn clean package -DskipTests

# -------- Stage 2: Run --------
FROM sapmachine:17-jdk AS run

# Set working directory for runtime container
WORKDIR /app

# Copy the built JAR file from the build stage
COPY --from=build /app/target/demo-0.0.1-SNAPSHOT.jar app.jar

# Expose the application port
EXPOSE 8080

# Run the application
CMD ["java", "-jar", "app.jar"]
```

---

## Steps to Build and Run

### Build the image

```bash
docker build -t app3_image .
```

### Run the container

```bash
docker run -d -p 8080:8080 --name app3_container app3_image
```

### Verify it’s running

```bash
curl http://localhost:8080
```

---

## Stop & Clean Up

Stop and remove the running container:

```bash
docker stop app3_container
docker rm app3_container
```

(Optional) Remove the image:

```bash
docker rmi app3_image
```

---

## Notes

* **Multi-stage builds** reduce the final image size by only including the JRE and the built JAR in the final image.
* The `mvn clean package -DskipTests` command speeds up the build process.
* Both stages use **SAPMachine 17**, ensuring consistent Java environments across build and runtime.

---

✅ **End of Task 3 – Multi-Stage Build for Java App**

### Screenshots
- **Running Application** ![App Screenshot](images/output.PNG)
