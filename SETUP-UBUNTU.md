# Ubuntu Setup Guide — Install Everything for This Course

This is a **one-time setup checklist** for a fresh Ubuntu machine (tested on Ubuntu 22.04 LTS and 24.04 LTS). Run through it once before starting Module 0, and you'll have every tool needed for all 11 modules already installed. Each section is copy-pasteable and includes a verification command.

> **Tip:** Run `sudo apt update && sudo apt upgrade -y` first if this is a fresh system.

## Quick Reference: What Each Module Needs

| Tool | Needed From |
|---|---|
| Terminal / bash | Module 0 |
| AWS CLI v2 | Module 1 onward (all modules) |
| Docker | Module 3 onward |
| `zip` / `unzip` | Module 3, 6, 9, 11 |
| `eksctl` | Module 4, 5, 7, 8, 10 |
| `kubectl` | Module 4, 5, 7, 8, 10 |
| `helm` | Module 4 |
| Node.js + npm | Module 5, 6, 9, 10, 11 |
| `jq` | Useful throughout for parsing JSON output |
| `git` | Module 8 |
| `redis-tools` (redis-cli) | Module 11 |
| `python3` | Module 4, 10 (small JSON parsing helpers) |

## 1. System Essentials

```bash
sudo apt update
sudo apt install -y curl wget unzip zip git jq python3 python3-pip build-essential ca-certificates gnupg lsb-release
```

**Verify:**
```bash
curl --version && unzip -v && git --version && jq --version && python3 --version
```

## 2. AWS CLI v2

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
rm -rf awscliv2.zip aws/
```

**Verify:**
```bash
aws --version
```

**Configure** (do this after installing — see Module 1, Lab 1.1 for the full account-setup walkthrough):
```bash
aws configure
# Region: us-east-1
```

## 3. Docker Engine

```bash
# Remove any old/conflicting versions first
sudo apt remove -y docker docker-engine docker.io containerd runc 2>/dev/null

# Set up Docker's official apt repository
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Allow running docker without sudo
sudo usermod -aG docker $USER
newgrp docker
```

**Verify** (log out and back in first if `docker` commands still ask for `sudo`):
```bash
docker --version
docker run hello-world
```

## 4. kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

**Verify:**
```bash
kubectl version --client
```

## 5. eksctl

```bash
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH
curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
sudo mv /tmp/eksctl /usr/local/bin
```

**Verify:**
```bash
eksctl version
```

## 6. Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

**Verify:**
```bash
helm version
```

## 7. Node.js + npm (via NodeSource, gives you a current LTS version)

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
```

**Verify:**
```bash
node --version
npm --version
```

## 8. redis-tools (for the `redis-cli` used in Module 11)

```bash
sudo apt install -y redis-tools
```

**Verify:**
```bash
redis-cli --version
```

## 9. (Optional but Recommended) A Code Editor

If you don't already have one:
```bash
sudo snap install code --classic
```
**Verify:** launch with `code .` from any folder.

## Full One-Shot Install Script

If you'd rather run everything at once, save this as `setup.sh` and run `bash setup.sh`:

```bash
#!/bin/bash
set -e

echo "=== 1. System essentials ==="
sudo apt update
sudo apt install -y curl wget unzip zip git jq python3 python3-pip build-essential ca-certificates gnupg lsb-release redis-tools

echo "=== 2. AWS CLI v2 ==="
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip -q awscliv2.zip
sudo ./aws/install
rm -rf awscliv2.zip aws/

echo "=== 3. Docker ==="
sudo apt remove -y docker docker-engine docker.io containerd runc 2>/dev/null || true
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo usermod -aG docker $USER

echo "=== 4. kubectl ==="
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

echo "=== 5. eksctl ==="
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH
curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
sudo mv /tmp/eksctl /usr/local/bin

echo "=== 6. Helm ==="
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

echo "=== 7. Node.js LTS ==="
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

echo "=== All done! Log out and back in (or run 'newgrp docker') for Docker group permissions to apply. ==="
echo "=== Next step: run 'aws configure' with your access key, secret key, and region 'us-east-1'. ==="
```

## Final Verification Checklist

Run this block after everything installs and after logging back in (for Docker group permissions):

```bash
echo "AWS CLI:   $(aws --version)"
echo "Docker:    $(docker --version)"
echo "kubectl:   $(kubectl version --client --short 2>/dev/null || kubectl version --client)"
echo "eksctl:    $(eksctl version)"
echo "Helm:      $(helm version --short)"
echo "Node.js:   $(node --version)"
echo "npm:       $(npm --version)"
echo "jq:        $(jq --version)"
echo "redis-cli: $(redis-cli --version)"
echo "git:       $(git --version)"
```

If every line prints a version number with no errors, you're fully set up for Module 0 through Module 11.

## Troubleshooting

- **`docker: permission denied` even after `usermod -aG docker $USER`** → You must fully log out and back in (or reboot) for group membership changes to take effect; `newgrp docker` works for the current terminal session only.
- **`kubectl`/`eksctl`/`helm` command not found after install** → Confirm `/usr/local/bin` is in your `PATH`: `echo $PATH`. It is by default on Ubuntu, but custom shell configs can override this.
- **AWS CLI install fails with "unzip: command not found"** → Run Section 1 (System Essentials) first — it installs `unzip`.
- **Apt-based Docker install fails on Ubuntu versions AWS/Docker haven't published packages for yet** → Check https://docs.docker.com/engine/install/ubuntu/ for the latest supported codenames, or use the fallback convenience script: `curl -fsSL https://get.docker.com | sudo sh`.
