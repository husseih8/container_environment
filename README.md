# Setting up the environment

So i wanted to get a bit of a native feel with running containers, so i've opted to using WSL which uses fewer system resources,
escpecially CPU and RAM and also get systemd !!

First thing is once you've installed wsl running Ubuntu https://learn.microsoft.com/en-us/windows/wsl/systemd 
You can check and set the default 
```
wsl -l -v 
wsl --set-default Ubuntu 
wsl -d Ubuntu 
```
You can then run the following to make sure systemd is running as PID 1, which is crucial in order to be able to 
be able to run docker as a service. 
```
ps -p 1 -o comm= 
```
This should show systemd as the top level PID
If not you may need to update your wsl, as per the doc mentioned above. 

Now, this is where it gets a little more interesting, 

Run the following 

1. Update packages - refreshing your local list of available packages
```
sudo apt update && sudo apt upgrade -y
```
2. Install dependencies - so these are helper tools, for fetching and verifyinf the docker packcages
so the certificates help verify identity of servers, so thats needed - and gnupg for handling the keys that the docker packages that are installed 
haven't been tampered with, and the release information about the base image(Ubuntu)
```
sudo apt install -y ca-certificates curl gnupg lsb-release
```
3. Add Docker’s official GPG key - where we will store dockers public gpg keys, and convert it from text 
to binary using --dearmor. This is what will be used to verify the authenticity of dockers software packages 
and then we make the key readable with the a+r so the package manager can use it. 
```
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```
4. Set up the Docker repo - adding dockers official package repository to the system, lsb_release here just returns
the codename for the version of Ubuntu you are running
```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
Results for this should look like the below or similar 
```
deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu jammy stable
```
line is written to docker.list this is for apt to know to use it when installing docker

5. Update package list again - so that apt index knows what packages docker offers, plus apt wouldn't 
know docker exists on the system otherwise
```
sudo apt update
```
6. Install Docker
Installing docker community edition and cli alongside containerd runtime used by docker under the hood 
to manage containers.
```
sudo apt install -y docker-ce docker-ce-cli containerd.io
```
7. Enable and start Docker (now works thanks to systemd 🎉)

This is the best part now, we can interact with docker as a service in our environment and you can also list 
more information about it by issuing sudo systemctl status 
```
sudo systemctl enable docker
sudo systemctl start docker
```
example output of the service:
```
State: running
    Units: 347 loaded (incl. loaded aliases)
     Jobs: 0 queued
   Failed: 0 units
    Since: Tue 2025-04-08 00:20:56 BST; 12h ago
  systemd: 255.4-1ubuntu8.6
   CGroup: /
           ├─init.scope
           │ ├─    1 /usr/lib/systemd/systemd --system --deserialize=50
```




Now you can confirm whether its working by running 
```
sudo docker version 
sudo run hello-world 
sudo info | grep -i cgroup 
```
This command should give you the below output 
```
 Cgroup Driver: systemd
 Cgroup Version: 2
  cgroupns
```
NOTE: always run docker with sudo for now but you can get rid of this by creating a docker group 
```
sudo groupadd docker 
```
Add your user to the docker group 
```
sudo usermod -aG docker $USER
```
Then run the below to apply it to the current session
```
newgrp docker 
```
You can then test it with 
```
docker ps -a 
```
This does give docker root-equivalent permissions on the system also, so not very feasible on a production environment 
as an attacker can break out of containers , modify root - and run arbitrary code on the host.

Now will install "Kubernetes in Docker" or for short kind
Tool is great for practicing some of the concepts you learn when you start learning k8s, and nice
for testing for local development for now.
```
cd ~
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.22.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

