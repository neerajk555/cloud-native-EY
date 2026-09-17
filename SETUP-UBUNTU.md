# Ubuntu Setup

These labs assume **Ubuntu 22.04+** (a real machine, a cloud VM, or
WSL2 running Ubuntu) with the AWS CLI and bash — not the Windows/
PowerShell path some earlier course materials used.

## 1. Install prerequisites

```bash
sudo apt update

# AWS CLI v2
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install -y unzip
unzip awscliv2.zip
sudo ./aws/install
rm -rf awscliv2.zip aws/

# Docker
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER
newgrp docker   # or log out/in for the group change to take effect

# kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
rm kubectl

# eksctl
curl --silent --location "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin

# Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Node.js (some Lambda labs)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# jq, git, zip/unzip
sudo apt install -y jq git zip unzip
```

## 2. Configure AWS CLI with your issued credentials

```bash
aws configure
# AWS Access Key ID:     (from the credentials your instructor gave you)
# AWS Secret Access Key: (same)
# Default region name:   us-east-1
# Default output format: json
```

Verify:

```bash
aws sts get-caller-identity
```

## 3. Export your participant variable (do this every new shell session)

```bash
export PARTICIPANT=$(aws iam get-user --query "User.UserName" --output text)
echo "You are: $PARTICIPANT"
```

Add the `export PARTICIPANT=...` line to `~/.bashrc` if you want it set
automatically in every new terminal:

```bash
echo 'export PARTICIPANT=$(aws iam get-user --query "User.UserName" --output text)' >> ~/.bashrc
```

Every lab in this repo uses `$PARTICIPANT` for resource names and the
`Owner` tag.

## 4. Verify Docker and kubectl

```bash
docker version
kubectl version --client
```

You're ready for Module 0 (optional CLI warm-up) or Module 1.
