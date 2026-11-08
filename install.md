# Docker, kubectl and Minikube Installation on Ubuntu

This guide installs Docker Engine, `kubectl`, and Minikube on Ubuntu.
These tools are required for the Kubernetes lab experiments:

-   Experiment 4: Deploy an Ubuntu container in Kubernetes
-   Experiment 5: Run a one-time BusyBox Pod
-   Experiment 6: Deploy Nginx, expose it using a Service, and scale the
    Deployment

> **Note:** The commands below assume a 64-bit Ubuntu system (`amd64`).
> Check your architecture with `dpkg --print-architecture`.

------------------------------------------------------------------------

## 1. Update Ubuntu

Open the Ubuntu Terminal and run:

``` bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y ca-certificates curl
```

## 2. Install Docker Engine

### Step 2.1: Remove conflicting older packages

The following removes older or conflicting Docker packages if they are
installed:

``` bash
sudo apt remove -y docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc
```

It is okay if Ubuntu reports that some of these packages are not
installed.

### Step 2.2: Create the Docker repository key directory

``` bash
sudo install -m 0755 -d /etc/apt/keyrings
```

### Step 2.3: Download Docker's official GPG key

``` bash
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

### Step 2.4: Add the official Docker APT repository

Create a script:

``` bash
vi docker.sh
```

Press `i` to enter insert mode, then paste the following complete
script:

``` bash
#!/bin/bash
set -e

sudo apt update
sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

sudo tee /etc/apt/sources.list.d/docker.sources > /dev/null <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
```

Save and exit `vi`:

1.  Press `Esc`.
2.  Type `:wq`
3.  Press `Enter`.

Make the script executable and run it:

``` bash
chmod +x docker.sh
./docker.sh
```

This script adds Docker's official repository and refreshes the package
list.

### Step 2.5: Install Docker Engine and plugins

Run this command after the script completes:

``` bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Step 2.6: Start Docker and enable it at boot

``` bash
sudo systemctl enable --now docker
```

Check the Docker version:

``` bash
docker --version
```

Test Docker:

``` bash
sudo docker run hello-world
```

You should see a message beginning with **Hello from Docker!**

### Step 2.7: Run Docker without typing `sudo` every time

``` bash
sudo usermod -aG docker $USER
```

Apply the group change in the current terminal:

``` bash
newgrp docker
```

Now test:

``` bash
docker run hello-world
docker ps
```

If you still get a permission error, log out of Ubuntu and log back in,
then try again.

> **Security note:** Membership in the `docker` group grants privileges
> equivalent to root access. Add only trusted users to this group.

------------------------------------------------------------------------

## 3. Install kubectl

`kubectl` is the command-line tool used to communicate with and manage a
Kubernetes cluster.

### Step 3.1: Update package information and install curl

``` bash
sudo apt update
sudo apt install -y curl
```

### Step 3.2: Download kubectl

``` bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

### Step 3.3: Install kubectl

``` bash
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

### Step 3.4: Verify the installation

``` bash
kubectl version --client
```

You should see the kubectl client version.

------------------------------------------------------------------------

## 4. Install Minikube

Minikube creates and runs a local, single-node Kubernetes cluster. In
this guide, it will use Docker as its driver.

### Step 4.1: Download Minikube

``` bash
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
```

### Step 4.2: Install Minikube

``` bash
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

### Step 4.3: Verify the installation

``` bash
minikube version
```

You should see the installed Minikube version.

------------------------------------------------------------------------

## 5. Start the Kubernetes cluster

Make sure Docker is running:

``` bash
sudo systemctl start docker
```

Start Minikube using the Docker driver:

``` bash
minikube start --driver=docker
```

The first startup may take several minutes while Minikube downloads the
required images.

### Verify the cluster

``` bash
kubectl get nodes
```

Expected output will be similar to:

``` text
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   ...   ...
```

The important point is that the node's status should be `Ready`.

Check cluster information:

``` bash
kubectl cluster-info
```

Check Minikube status:

``` bash
minikube status
```

------------------------------------------------------------------------

## 6. Useful commands for Experiments 4, 5 and 6

  Purpose                     Command
  --------------------------- ---------------------------------
  Start the cluster           `minikube start`
  Stop the cluster            `minikube stop`
  Check Minikube status       `minikube status`
  View Kubernetes nodes       `kubectl get nodes`
  View Pods                   `kubectl get pods`
  View Deployments            `kubectl get deployments`
  View Services               `kubectl get services`
  View all common resources   `kubectl get all`
  View Pod details            `kubectl describe pod POD_NAME`
  View Pod logs               `kubectl logs POD_NAME`

**Important:** Install these tools only once. Reuse the same Minikube
cluster for Experiments 4, 5 and 6. You do not need to delete the
cluster between experiments.

------------------------------------------------------------------------

## 7. Troubleshooting

### Docker service is not running

``` bash
sudo systemctl start docker
sudo systemctl status docker
```

### Docker permission denied

``` bash
sudo usermod -aG docker $USER
newgrp docker
docker ps
```

If the issue continues, log out and log back in.

### Minikube does not start

Check Docker first:

``` bash
docker ps
```

Then try:

``` bash
minikube start --driver=docker
```

Check its status:

``` bash
minikube status
```

### A Pod is not starting

``` bash
kubectl get pods
kubectl describe pod POD_NAME
kubectl logs POD_NAME
```

Replace `POD_NAME` with the actual Pod name shown by `kubectl get pods`.

------------------------------------------------------------------------

## 8. Final installation checklist

Run these commands to confirm that the tools are installed:

``` bash
docker --version
kubectl version --client
minikube version
kubectl get nodes
```

Installation is ready for the Kubernetes lab experiments when the
commands return version information and the Minikube node shows `Ready`.
