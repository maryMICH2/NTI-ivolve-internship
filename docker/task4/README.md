# Managing Docker Environment Variables Across Build and Runtime

## Task Overview

This project demonstrates how to manage environment variables in Docker for different environments — **development**, **staging**, and **production** — using **Python** and **Flask**.

You will:

* Use a Python base image
* Install Flask
* Configure environment variables across build and runtime
* Run the container with different environment settings

---

## Clone the Application Code

```bash
git clone https://github.com/Ibrahim-Adel15/Docker-3.git
cd Docker-3
```

---

## Write Dockerfile

```dockerfile
FROM python:3.10-slim

# Default (production) environment variables
ENV APP_MODE=production
ENV APP_REGION=canada-west

# Set working directory
WORKDIR /app

# Copy project files into the container
COPY . .

# Install Flask
RUN pip install --no-cache-dir flask

# Expose Flask app port
EXPOSE 8080

# Command to run the application
CMD ["python", "app.py"]
```

---

 
##  Create `env` File for Staging

Create a file named `env` in the same directory:

```
APP_MODE=staging
APP_REGION=us-west
```

---

## Build Docker Image

```bash
docker build -t flask-env-image .
```

---

## Run Containers in Different Modes

### 🔹 Development Mode (Set Variables Inline)

```bash
docker run -d -p 8080:8080 \
-e APP_MODE=development \
-e APP_REGION=us-east \
flask-env-image
```

### 🔹 Staging Mode (Using `env` File)

```bash
docker run -d -p 8081:8080 --env-file env flask-env-image
```

### 🔹 Production Mode (Defaults from Dockerfile)

```bash
docker run -d -p 8082:8080 flask-env-image
```

---

## Test the Application

Check each version using `curl`:

```bash
curl localhost:8080   # → App mode: development, Region: us-east
curl localhost:8081   # → App mode: staging, Region: us-west
curl localhost:8082   # → App mode: production, Region: canada-west
```

---

## Verify Running Containers

```bash
docker ps
```

---

## Cleanup (Optional)

```bash
docker stop $(docker ps -aq)
docker rm $(docker ps -aq)
```

---

## Expected Output Examples

| Mode        | Command               | Output                                    |
| ----------- | --------------------- | ----------------------------------------- |
| Development | `curl localhost:8080` | App mode: development, Region: us-east    |
| Staging     | `curl localhost:8081` | App mode: staging, Region: us-west        |
| Production  | `curl localhost:8082` | App mode: production, Region: canada-west |

---

## Summary

| Environment | Set Where              | Variables                                   |
| ----------- | ---------------------- | ------------------------------------------- |
| Development | Inline in `docker run` | APP_MODE=development, APP_REGION=us-east    |
| Staging     | `.env` file            | APP_MODE=staging, APP_REGION=us-west        |
| Production  | Dockerfile             | APP_MODE=production, APP_REGION=canada-west |

