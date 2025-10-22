# Docker Volume and Bind Mount with Nginx

## Objective

Use Docker volumes and bind mounts to manage persistent Nginx logs and serve custom HTML content from the host machine.

---

## Steps

### 1. Create a Volume for Logs

Create a Docker volume to store Nginx logs persistently.

```bash
docker volume create nginx_logs
```

Verify the volume:

```bash
docker volume inspect nginx_logs
```

Screenshot: **mountpoint-for-the-created-volume**
[mountpoint](images/mountpoint-for-the-created-volume.png)

---

### 2. Create a Bind Mount Directory

Create a directory on your host to serve as the bind mount.

```bash
mkdir -p nginx-bind/html
```

Create an `index.html` file inside it:

```bash
echo "Hello from Bind Mount" > nginx-bind/html/index.html
```

Screenshot: **html-outside-the-container**
[html-outside-the-container](images/html-outside-the-container.png)

---

### 3. Run Nginx Container with Volume and Bind Mount

```bash
docker run -d \
  --name nginx_test \
  -p 8080:80 \
  -v nginx_logs:/var/log/nginx \
  -v "$(pwd)/nginx-bind/html":/usr/share/nginx/html:ro \
  nginx:latest
```

Verify container is running:

```bash
docker ps
```

Screenshot: **html-inside-the-container**
[html-inside-the-container](images/html-inside-the-container.png)
---

### 4. Test the Application

Use `curl` or a browser to verify the page:

```bash
curl localhost:8080
```

You should see:

```
Hello from Bind Mount
```

---

### 5. Update HTML and Verify Changes

Update the file on the host:

```bash
echo "Updated text: Hello from NTI task!" > nginx-bind/html/index.html
```

Screenshot: **updated-html-outside**
[updated-html-outside-the-container](images/updated-html-outside.png)
Then, test again:

```bash
curl localhost:8080
```

You should now see the updated text.

Screenshot: **updated-html-inside**
[updated-html-inside-the-container](images/updated-html-inside.png)
---

### 6. Verify Logs

Check Nginx logs stored in the volume:

```bash
sudo ls /var/lib/docker/volumes/nginx_logs/_data
sudo tail -n 10 /var/lib/docker/volumes/nginx_logs/_data/access.log
```

---

### 7. Clean Up

Stop and remove the container:

```bash
docker stop nginx_test
docker rm nginx_test
```

Remove the volume:

```bash
docker volume rm nginx_logs
```

---

## Summary

This task demonstrates:

* How to use Docker volumes for persistent log storage.
* How to use bind mounts to serve and dynamically update web content from the host.

