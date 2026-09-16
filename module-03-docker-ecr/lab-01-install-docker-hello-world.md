# Lab 3.1: Install Docker & Run Hello World

**Module:** 3 — Containerization with Docker & Amazon ECR
**Difficulty:** Beginner
**Duration:** ~15 minutes
**Cost:** ✅ Free (all local, no AWS resources)

## Objective
Install Docker on your machine and run your first container, confirming your environment is ready for the rest of the module.

## Prerequisites
- A Windows, Mac, or Linux machine with admin rights to install software

## Steps

1. **Download and install Docker Desktop** (Windows/Mac) or **Docker Engine** (Linux):
   - https://www.docker.com/products/docker-desktop/
   - Follow the installer prompts for your OS. On Linux, you can alternatively run:
     ```bash
     curl -fsSL https://get.docker.com -o get-docker.sh
     sudo sh get-docker.sh
     ```
2. **Start Docker Desktop** (Windows/Mac) and wait for the whale icon in your system tray/menu bar to show "Docker is running."
3. **Verify the installation:**
   ```bash
   docker --version
   docker info
   ```
4. **Run the official hello-world container:**
   ```bash
   docker run hello-world
   ```
5. **List running/stopped containers:**
   ```bash
   docker ps -a
   ```

## Expected Result / Validation
- `docker run hello-world` should print a message starting with:
  ```
  Hello from Docker!
  This message shows that your installation appears to be working correctly.
  ```
- `docker ps -a` should show the `hello-world` container with `Exited (0)` status — meaning it ran successfully and completed.

## Cleanup
Remove the hello-world container and image so your local environment stays clean:
```bash
docker rm $(docker ps -aq --filter ancestor=hello-world)
docker rmi hello-world
```
Verify:
```bash
docker images
```
`hello-world` should no longer be listed.

## Troubleshooting
- **`Cannot connect to the Docker daemon`** → Docker Desktop isn't running yet; start it and wait ~30 seconds.
- **Permission denied on Linux** → Add your user to the docker group: `sudo usermod -aG docker $USER`, then log out/in.
