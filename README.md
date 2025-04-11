### Setting up the environment

So I wanted to get a bit of a native Linux feel when running containers, and I decided to go with WSL (since it's the only viable option while I'm on Windows). This setup ultimately uses fewer system resources—especially CPU and RAM—and also supports systemd, which is a huge win compared to using Podman or Docker Desktop.

---

### Setting up WSL with systemd support

First thing is, once you've installed WSL running Ubuntu:  
🔗 https://learn.microsoft.com/en-us/windows/wsl/systemd

You can check and set the default distro with:

```bash
wsl -l -v 
wsl --set-default Ubuntu 
wsl -d Ubuntu 
```

You can then run the following to make sure systemd is running as PID 1, which is crucial in order to be able to run Docker as a service:

```bash
ps -p 1 -o comm=
```

> This should show `systemd` as the top-level PID. If not, you may need to update your WSL setup, as per the docs linked above.

---

### Installing Docker (with systemd support)

Now, this is where it gets a little more interesting:

#### 1. **Update packages** – Refreshing your local list of available packages:

```bash
sudo apt update && sudo apt upgrade -y
```

#### 2. **Install dependencies** – Helper tools for fetching and verifying Docker packages:

```bash
sudo apt install -y ca-certificates curl gnupg lsb-release
```

> - `ca-certificates`:  This will verify the Docker packages and the identity of the servers.
> - `gnupg`: GPG is useful for handling the keys that the Docker packages that are installed haven't been tampered with.  
> - `lsb-release`: This gives you the release name of the Ubuntu version you are running.


#### 3. **Add Docker’s official GPG key**:

```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```


#### 4. **Set up the Docker repo**:

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

> Example output:
> You can view this file manually also to see if its been appropriately populated /etc/apt/sources.list.d/docker.list.
> `deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu jammy stable`

#### 5. **Update apt again** – To include Docker in the list of packages:

At this point we want to update the APT index so that it knows what packages docker offers, if this is not run, apt wouldn't know if docker exists on the system.
```bash
sudo apt update
```

#### 6. **Install Docker Community Edition and container runtime**:

```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io
```

---

### ⚙️ Enable and start Docker with systemd

Now this is the best part—interacting with Docker as a proper service in your WSL environment:

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

#### You can check Docker’s status:

```bash
sudo systemctl status docker
```

Example snippet of output:

```
State: running
Units: 347 loaded
Jobs: 0 queued
Failed: 0 units
Since: Tue 2025-04-08 00:20:56 BST; 12h ago
systemd: 255.4-1ubuntu8.6
CGroup: /
       ├─init.scope
       │ ├─    1 /usr/lib/systemd/systemd
```

---

### Test Docker

```bash
sudo docker version
sudo docker run hello-world
sudo docker info | grep -i cgroup
```

> Expected output snippet:
```
Cgroup Driver: systemd
Cgroup Version: 2
cgroupns
```

---

### Optional: Run Docker without `sudo`

1. Create the Docker group (if it doesn’t already exist):

```bash
sudo groupadd docker
```

2. Add your user to the group:

```bash
sudo usermod -aG docker $USER
```

3. Apply group changes to current session:

```bash
newgrp docker
```

4. Test:

```bash
docker ps -a
```

> Note: Docker group gives root-equivalent permissions. Avoid using this in production as it may allow attackers to escape containers and access the host system.

---

### Install Kind (Kubernetes in Docker)

Kind is a great tool for making a lightweight Kubernetes setup, I explore this more deeply in my falco-playground.

```bash
cd ~
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.22.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

---

### ➕ Additional Useful Steps

Here are a few extra steps you could consider adding to your setup:

#### Enable Docker to start automatically on WSL launch

Append to your `~/.bashrc` or `~/.zshrc`:

```bash
sudo systemctl start docker >/dev/null 2>&1
```

You can make this smoother with a conditional check to avoid errors if Docker is already running.

#### Install Docker Compose (optional but useful) 

I have a repo where I explore using docker-compose to emulate an environment with an adversary container.

```bash
sudo apt install docker-compose-plugin
docker compose version
```

#### 🧹 Clean up unused Docker resources (optional)

```bash
docker system prune -a
```


