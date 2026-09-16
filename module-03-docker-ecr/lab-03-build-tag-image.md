# Lab 3.3: Build & Tag a Docker Image

**Module:** 3 — Containerization with Docker & Amazon ECR
**Difficulty:** Beginner
**Duration:** ~15 minutes
**Cost:** ✅ Free (all local)

## Objective
Build the Dockerfile from Lab 3.2 into a runnable image, tag it following semantic versioning conventions, and run it locally.

## Prerequisites
- Completed Lab 3.2 (the `hello-cloudnative-app` folder with its Dockerfile)

## Steps

1. **Navigate to the project folder:**
   ```bash
   cd hello-cloudnative-app
   ```
2. **Build the image**, tagging it with a version:
   ```bash
   docker build -t hello-cloudnative-app:1.0.0 .
   ```
3. **Also tag it as `latest`** (common convention for the most recent build):
   ```bash
   docker tag hello-cloudnative-app:1.0.0 hello-cloudnative-app:latest
   ```
4. **List your images and inspect layers:**
   ```bash
   docker images | grep hello-cloudnative-app
   docker history hello-cloudnative-app:1.0.0
   ```
5. **Run a container from the image**, mapping port 3000:
   ```bash
   docker run -d -p 3000:3000 --name hello-app-container hello-cloudnative-app:1.0.0
   ```
6. **Test it:**
   ```bash
   curl http://localhost:3000
   ```

## Expected Result / Validation
- `curl http://localhost:3000` should return:
  ```
  Hello from a cloud native container!
  ```
- `docker images` should show two tags (`1.0.0` and `latest`) pointing to the same Image ID (confirming they're the same underlying image).

## Cleanup
Stop and remove the running container, then remove the images:
```bash
docker stop hello-app-container
docker rm hello-app-container
docker rmi hello-cloudnative-app:1.0.0 hello-cloudnative-app:latest
```
Verify:
```bash
docker ps -a
docker images | grep hello-cloudnative-app
```
Both should return no results for this app.

## Troubleshooting
- **Port already in use** → Another process is using 3000; either stop it or run with `-p 3001:3000` and test `http://localhost:3001` instead.
- **Build fails on `npm install`** → Confirm you're in the correct directory with `package.json` present (`ls`).
