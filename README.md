# SRE hands-on project for including these tasks:
- [Task 0: Preparation for the Interview Environment](#task-0-preparation-for-the-interview-environment)
- [Task 1: SSH & Upgrade Operating System](#task-1-ssh--upgrade-operating-system)
- [Task 2: Install Docker Engine](#task-2-install-docker-engine)
- [Task 3: Install Gogs (a self-hosted git service)](#task-3-install-gogs-a-self-hosted-git-service)
- [Task 4: Create a Demo Organization/Repository in Gogs](#task-4-create-a-demo-organizationrepository-in-gogs)
- [Task 5: Install a Single Node Kubernetes Cluster Using Kind](#task-5-install-a-single-node-kubernetes-cluster-using-kind)
- [Task 6: Build and Run a Golang Web Application](#task-6-build-and-run-a-golang-web-application)
- [Task 7: Run the App in Container](#task-7-run-the-app-in-container)
- [Task 8: Deploy the Hello World Container Image in Kubernetes](#task-8-deploy-the-hello-world-container-image-in-kubernetes)
- [Task 9: Install Kubernetes Dashboard](#task-9-install-kubernetes-dashboard)
- [Task 10: Build Gogs container image and push it to a container registry](#task-10-build-gogs-container-image-and-push-it-to-a-container-registry)
- [Task 99: Publish your work](#task-99-publish-your-work)

## **Task 0: Preparation for the Interview Environment**

The candidate shall prepare the following environment for the interview:

*   A Laptop or Desktop capable of running Virtualization and with an ssh client available to use. Any platforms (Windows, MacOS or Linux) are good.
*   A good internet connection capable of pulling system packages and application packages from the internet without significant delay.
*   A container registry account which can push and pull container images in a publicly accessible container registry
    * For external candidate, suggest to use dockerHub account (free tier is ok), [https://hub.docker.com](https://hub.docker.com) 
    * For internal candidate, please use ARM docker ([armdocker.rnd.ericsson.se](https://armdocker.rnd.ericsson.se)) rather than external registeries to avoid any security concerns
*   Create a virtual machine (guest machine) on localhost (host machine) and install the latest **Ubuntu 24.04 server 64-bit** version.
    *   Can choose any hypervisor (virtualbox, vmware workstation/fusion, kvm, hyper-v, …).
      *   Use virtualbox as a quick start for non-Apple silicons: [https://www.virtualbox.org/](https://www.virtualbox.org/).
      *   Windows Subsystem Linux (WSL 1 or 2) may not satisfy the requirement.
      *   Alternatively, you can also use any turn-key VM solution to spawn up a Ubuntu Linux, e.g. [multipass](https://multipass.run/), [orbstack](https://orbstack.dev/), so long as you can satisfy the requirement.
    *   4 x vCPUs and 8GB RAM preferred / 2 x vCPUs and 4GB RAM minimum
    *   Use NAT for VM network
    *   Ubuntu distribution: [https://releases.ubuntu.com/24.04/](http://releases.ubuntu.com/22.04/)
    *   sshd must be installed and enabled for ssh access
    *   docker/container runtime and golang can be pre-installed to save time

 ### Setup the Ubuntu Server VM on the Windows laptop:
1. Download and install the hypervisor VirtualBox: https://www.virtualbox.org/
2. Download the latest Ubuntu Server 64-bit version: https://releases.ubuntu.com/noble/
3. Create the VM:
   - Type VM Name
   - Attach the downloaded Ubuntu Server ISO file
   - Setup username and password
   - Specify virtual hardware: 4 x vCPUs, 8GB RAM and 40GB for the disk
   - Choose NAT for VM network in VM -> Settings -> Network
    
---

All the following tasks are to be performed in the ubuntu virtual machine, unless stated otherwise.

## **Task 1: SSH & Upgrade Operating System**

1. Install and enable SSH for remote login
   ```bash
   sudo apt update
   sudo apt install -y openssh-server
   sudo systemctl enable ssh
   sudo systemctl start ssh
   ```
2. Setup NAT port forwarding to enable SSH from host:
   <br>In VirtualBox -> Settings -> Network -> Port Forwarding, add rule:
      - Name: SSH
      - Protocol: TCP
      - Host IP: 127.0.0.1
      - Host Port: 2222
      - Guest Port: 22
        
   <br>Then open PowerShell on the host and SSH into the VM using:
      ```bash
      ssh wangqinglhc0122@127.0.0.1 -p 2222
      ```
   enter password and then you will log into the VM
3. Setup the public key authentification for the host to SSH into the VM:
   - In host PowerShell, do:
      ```bash
      ssh-keygen -t ed25519
      ```
   - This creates:
      - Public key: ~/.ssh/id-ed25519
      - Private key: ~/.ssh/id_ed25519.pub
   - Copy the contents in the public key, SSH into the VM using password, and paste the contents into ~/.ssh/authorized_keys in the VM
   - Then, in the host PowerShell, you can SSH into the VM without password anymore:
      ```bash
      ssh wangqinglhc0122@127.0.0.1 -p 2222
      ```
4. Upgrade the Ubuntu system packages:
   <br>In the VM:
      ```bash
      sudo apt update
      sudo apt full-upgrade -y
      sudo apt autoremove -y
      ```
5. Upgrate the Linux kernel to the latest LTS Enablement for Hardware Enablement (HWE) stack level:
   - Check the kernel version:
      ```bash
      uname -r
      ```
   - Upgrade to the latest HWE kernel for Ubuntu 24.04 LTS:
      ```bash
      sudo apt install --install-recommends linux-generic-hwe-24.04
      ```
   - Reboot the VM and then check the kernel version:
      ```bash
      uname -r
      ```
---      
_Q: What is host machine and guest machine?_
<br>A: The host machine is the physical computer, my Windows laptop, which runs the hypervisor.
<br>The guest machine is the VM running Ubuntu inside the hypervisor.

_Use an ssh client in the host machine to ssh to the guest machine using public key authentication method._

_Q: What is the IP of the guest machine? And is it accessible from the host machine? What configuration is required in order to have ssh access?_
<br>A: The IP of the guest machine can be viewed using:
```bash
ip a
```
and it is 10.0.2.15. Beacuse we are using NAT for the network, the host can not access the guest via its IP. We need to use Port Forwarding.

_Q: What are the difference between NAT and Bridge network?_
<br>A: For NAT, the guest get its private IP which is hidden from LAN and port forwarding is needed for the host to access it.
<br>For bridged network, the guest get real LAN IP and it is directly accessible to the host.

_Q: How does the public key authentication method work? Will your private key travel to the remote server for authentication?_
<br>A: The public key authentication method generates a pair of PUBLIC and PRIVATE keys. The client signs chellenge using PRIVATE key and the server verifies signature using PUBLIC key. The PRIVATE key does not travel to the remote server.

_After ssh via client terminal, upgrade the **Ubuntu system packages (packages pre-installed in the system)** to the latest ([https://ubuntu.com/server/docs/package-management](https://ubuntu.com/server/docs/package-management))._

_Q: Figure out what the **linux kernel version** is and then upgrade the kernel to the **latest LTS Enablement or Hardware Enablement (HWE) stack kernel ([https://ubuntu.com/kernel/lifecycle](https://ubuntu.com/kernel/lifecycle)). What is the kernel version after upgrade?_
<br>A: The kernel version after upgrade is:
```bash
6.17.0-14-generic
```

## **Task 2: Install Docker Engine**

Install Docker Engine from Docker maintained repository (not Ubuntu maintained repository) [https://docs.docker.com/install/linux/docker-ce/ubuntu/](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

Install and setup Docker:
   - Install and enable Docker:
      ```bash
      sudo apt update
      sudo apt install -y docker.io
      sudo systemctl enable docker
      sudo systemctl start docker
      ```
   - Add user to run docker without sudo:
      ```bash
      sudo usermod -aG docker $USER
      ```
   - Run docker hello world to verify the setup:
      ```bash
      docker run hello-world
      ```
      
_Q: What are the relationships of the three components in Docker Engine, aka. docker-ce docker-ce-cli and containerd.io?_
<br>
<br>A: Docker-ce installs dockerd, the Docker Daemon which is like the brain of Docker, it listens to the docker-ce-cli requests, manages the images and networks and talks to containerd.
<br>Docker-ce-cli is the remote control which sends HTTP requests to the Docker Daemon.
<br>Containerd.io is the runtime manager, which pulls images and magage container lifecycle, manages low-level container operations. It is used by Kubernetes

## **Task 3: Install Gogs (a self-hosted git service)**

Install gogs from docker: [https://github.com/gogs/gogs/tree/main/docker](https://github.com/gogs/gogs/tree/main/docker) 

Choose the deployment configuration you prefer, just make sure it listens to port 3100 in the guest machine.

Expect output: Gogs is up and running at the guest machine, listening to port 3100. How to check if it is up and running as expected?

Open a browser in host machine and access gogs via [http://localhost:3100](http://localhost:3100)

1. Create gogs directory:
   ```bash
   mkdir -p ~/gogs
   cd ~/gogs
   ```
2. Run gogs container with:
   ```bash
   docker run -d --name=gogs -p 3100:3000  -p 10022:22 -v ~/gogs:/data --restart=unless-stopped gogs/gogs
   ```
   - Guest port 3100 -> container port 3000
   - SSH ports: guest port 10022 -> container port 22
   - DB stored in /data
   - Persistent volume mounted at ~/gogs
3. Setup Port Forwarding to let the host access gogs:
  <br>In VirtualBox -> Settings -> Network -> Port Forwarding, add rule:
   - Name: Gogs
   - Protocol: TCP
   - Host IP: 127.0.0.1
   - Host Port: 3100
   - Guest Port: 3100
   <br>Now the host can access gogs via:
      ```bash
      http://localhost:3100/
      ```
4. Configure gogs:
   - Database Type: SQLite3
   - Path: data/gogs.db
   - Domain: localhost
   - Application URL: http://localhost:3100/
   - SSH Port: 22
   - HTTP Port: 3100
   - Create admin user:
      - Username: root
      - Password: 123456
      - Email: root@example.com
5. Add SSH key to Gogs:
   - Copy SSH key contents in:
      ```bash
      ~/.ssh/id_rsa.pub
      ```
   - In Gogs -> Settings -> SSH Keys -> Add key, create a SSH key and paste the contents

_Q:Is the port 3100 of the guest machine accessible from the host machine? If not, what configuration is required in order to expose it?_
<br>
<br>A: No. We need to setup the Port Forwarding following step 3 above.

For the first time, configure it following the installation steps 

*   Database uses SQLite3 (no external DB required)
*   Figure out the domain, ports and URL to be used
*   Create an admin user as well (username: root password: 123456)

## **Task 4: Create a Demo Organization/Repository in Gogs**

Sign in to gogs using admin account and create a demo org/repo named demo/go-web-hello-world (demo is organization name, go-web-hello-world is repository name)

Initiate the repo with a README.md and push the commit to master

Expected output: [http://localhost:3100/demo/go-web-hello-world](http://localhost:3100/demo/go-web-hello-world) 

1. On the host, open in web browser:
   ```bash
   http://localhost:3100
   ```
   Login using:
      - Username: root
      - Password: 123456
2. In Dashboard -> Organizations, click + and create a orgnization called demo
3. Then create a repository inside demo called go-web-hello-world
4. In the VM, clone the repository:
   ```bash
   git clone ssh://git@localhost:10022/demo/go-web-hello-world.git
   ```
5. Create README.md file and commit it to master branch:
   ```bash
   git add README.md
   git commit -m "Add README"
   git push origin master
   ```
## **Task 5: Install a Single Node Kubernetes Cluster Using Kind**

Follow instruction at [https://kind.sigs.k8s.io/docs/user/quick-start/](https://kind.sigs.k8s.io/docs/user/quick-start/).

Install kind first (use the stable binary rather than build from source to save time), then create a single node Kubernetes cluster using it.

Install kubectl client to interact with the cluster [https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/) 

Get cluster info, node info and list all pods in the cluster.

Check-in the kubeconfig file (~/.kube/config) into the gogs git repo. 

---

1. Install kind using stable binary:
   ```bash
   curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.22.0/kind-linux-amd64
   chmod +x ./kind
   sudo mv ./kind /usr/local/bin/kind
   kind version
   ```
2. Install kubectl:
   ```bash
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
   chmod +x kubectl
   sudo mv kubectl /usr/local/bin/
   kubectl version --client
   ```
3. Create a single node Kubernetes cluster using kind:
   ```bash
   kind create cluster --name demo-cluster
   kubectl cluster-info
   kubectl get nodes -o wide
   ```
4. List all pods:
   ```bash
   kubectl get pods -A
   ```
5. Check the ~/.kube/config file to the gogs git repo:
   ```bash
   cp ~/.kube/config kubeconfig
   git add kubeconfig
   git commit -m "Add kubeconfig for kind cluster"
   git push origin master
   ```

Final architecture: Windows Host -> VirtualBox NAT -> Ubuntu VM -> Docker -> Kind -> Kubernetes single-node cluster

## **Task 6: Build and Run a Golang Web Application**

Use golang to build a hello world web app (listen to 8081 port) and check-in the code to the master branch of the repo created above

[https://golang.org/](https://golang.org/)

[https://gowebexamples.com/hello-world/](https://gowebexamples.com/hello-world/)

Expect source code at [http://localhost:3100/demo/go-web-hello-world](http://localhost:3100/demo/go-web-hello-world)

Expect output (from guest machine):
```
curl http://localhost:8081
Go Web Hello World!
```
---
1. In the VM, go to the repo:
   ```bash
   cd ~/go-web-hello-world
   ```
2. Create a main.go file with:
   ```Go
   package main

   import (
   	"fmt"
   	"log"
   	"net/http"
   )
   
   func main() {
   	mux := http.NewServeMux()
   	mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
   		fmt.Fprintln(w, "Hello, World!")
   	})
   
   	addr := ":8081"
   	log.Printf("Listening on %s\n", addr)
   	log.Fatal(http.ListenAndServe(addr, mux))
   }
   ```
3. Add a go.mod to make it a real module:
   ```bash
   go mod init demo/go-web-hello-world
   go mod tidy
   ```
4. Buid with Go:
   ```bash
   go build -o go-web-hello-world .
   ```
   and it will produce a runable binary: go-web-hello-world
5. Run the binary and verify output:
   ```bash
   ./go-web-hello-world
   ```
   it will output: Listening on :8081
   <br>Open another PowerShell on host and SSH into VM and verify the output:
   ```bash
   ssh wangqinglhc0122@127.0.0.1 -p 2222
   curl -s http://127.0.0.1:8081/
   ```
   it should print: Hello, World!
6. Check in code to the gogs repo master branch, add .gitignore file:
   ```bash
   git status
   echo "go-web-hello-world" >> .gitignore
   git add main.go go.mod .gitignore
   git commit -m "Add Go hello world web app on 8081"
   git push origin master
   ```
## **Task 7: Run the App in Container**

Build a docker image ($ docker build) for the web app, tag it go-app:latest ($ docker tag) and run it in a container ($ docker run), expose the service to 8082 (-p)

[https://docs.docker.com/engine/reference/commandline/build/](https://docs.docker.com/engine/reference/commandline/build/)

Check in the Dockerfile into git repo

Expect output (from guest machine):
```
curl http://localhost:8082
Go Web Hello World!
```

---
1. In the repo root ~/go-web-hello-world, create Dockerfile with:
   ```dockerfile
   FROM golang:1.22

   WORKDIR /app
   COPY . .
   
   RUN go mod tidy
   RUN go build -o server .
   
   EXPOSE 8081
   CMD ["/app/server"]
   ```
2. Build the image using tag: go-app:latest:
   ```bash
   docker build -t go-app:latest .
   ```
3. Run and expose to port 8082:
   ```bash
   docker run --rm -p 8082:8081 --name go-app go-app:latest
   ```
4. Verify from another terminal in VM:
   ```bash
   curl -s http://localhost:8082
   ```
   it should print: Hello, World!
5. Check in the Dockerfile to git:
   ```bash
   git add Dockerfile main.go
   git commit -m "Add Dockerfile to run Go web app in container"
   git push origin master
   ```
6. Check the image size:
   ```bash
   docker images | grep go-app
   ```
   the size is 1.32GB
7. Optimized the image size by creating Dockerfile.optimized with:
   ```dockerfile
   # ---- build stage ----
   FROM golang:1.22 AS build
   WORKDIR /src
   
   COPY go.mod ./
   # go.sum might not exist; that's fine
   COPY . .
   
   # Build a small static binary
   RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 \
       go build -trimpath -ldflags="-s -w" -o /out/server .
   
   # ---- runtime stage ----
   FROM scratch
   COPY --from=build /out/server /server
   EXPOSE 8081
   ENTRYPOINT ["/server"]
   ```
   and build the image:
   ```bash
   docker build -f Dockerfile.optimized -t go-app:optimized .
   ```
   The optimized image has size 7MB.
8. Check the Dockerfile.optimized to git:
   ```bash
   git add Dockerfile.optimized
   git commit -m "Add optimized multi-stage Dockerfile (<10MB scratch image)"
   git push origin master
   ```
_Q: What is the size of the container image? Can it be optimized? If yes, how? Can you please make it less than 10MB? Check in the optimized Dockerfile into git repo._
<br>
<br>A: The size is 1.32GB. We can optimize using multi-stage build and a scratch final image with a static, stripped binary. The optimized size is 7MB.



## **Task 8: Deploy the Hello World Container Image in Kubernetes**

Deploy the container image in the single node kubernetes created above and expose the service via nodePort 31080.

Figure out the node ip and access the service.

Expect output (from guest machine):
```
curl http://<node ip>:31080
Go Web Hello World!
```

Check in the deployment yaml file or the command line into the git repo

---
1. Load the go-app:optimized image to kind:
   ```bash
   kind load docker-image go-app:optimized --name demo-cluster
   ```
2. Crete Kubernetes manifests for deployment and service NodePort 31080:
   ```bash
   cd ~/go-web-hello-world
   mkdir -p k8s
   ```
   create k8s/go-web-hello-world.yaml with:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: go-web-hello-world
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: go-web-hello-world
     template:
       metadata:
         labels:
           app: go-web-hello-world
       spec:
         containers:
           - name: app
             image: go-app:optimized
             imagePullPolicy: IfNotPresent
             ports:
               - containerPort: 8081
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: go-web-hello-world
   spec:
     type: NodePort
     selector:
       app: go-web-hello-world
     ports:
       - name: http
         port: 8081
         targetPort: 8081
         nodePort: 31080
   ```
3. Apply it:
   ```bash
   kubectl apply -f k8s/go-web-hello-world.yaml
   ```
   wait it to be ready:
   ```bash
   kubectl rollout status deploy/go-web-hello-world
   kubectl get pods -o wide
   kubectl get svc go-web-hello-world
   ```
4. Get the node IP:
   ```bash
   kubectl get nodes -o wide
   ```
   The node IP is: 172.18.0.2
5. Access the service:
   ```bash
   curl -s http://172.18.0.2:31080
   ```
   it prints: Go Web Hello World!
8. Check the deplyment yaml to git:
   ```bash
   git add k8s/go-web-hello-world.yaml
   git commit -m "Add Kubernetes deployment and NodePort service (31080) for Go app"
   git push origin master
   ```
## **Task 9: Install Kubernetes Dashboard**

Install Headlamp (https://headlamp.dev/) in the cluster using Yaml manifest method (not helm) and expose the service to nodeport 31081.

Can you access the dashboard URL via your host machine (not vm guest machine)? If not, how to configure the network to achieve it?

Open a browser in host machine and access the Dashboard via: [https://localhost:31081](https://localhost:31081) (asking for token).

Figure out how to generate a token to login to the dashboard.

Publish the procedure to the git repo.

---
1. 


## **Task 10: Build Gogs container image and push it to a container registry**

Document all technical procedures above in a README.md file, and check it in, together with all text files created above to the gogs git repo. Please make sure you properly use the Markdown syntax for a well formatted technical procedure, including but not limited to Titles, Sub-titles, Code Block, Link, List and etc.

Please do not write the procedure in Word or any document other than Markdown. Please do not attach any screen copies either.

Build a container image for gogs with your gogs config and demo/go-web-hello-world repo embedded. Please figure out how to persist your gogs config and repo in docker container ([https://docs.docker.com/storage/](https://docs.docker.com/storage/)) to avoid an empty gogs app requiring initiation. Please document the procedure for the image build/push in the README.md file as well.

For non-Ericsson workforces:
- Use [dockerhub](https://hub.docker.com/)
- Tag the container image using your_dockerhub_id/gogs:v0.1 and push it to docker hub ([https://hub.docker.com/](https://hub.docker.com/))
- Create an overview file in your dockerhub repo for how to pull the image and start the gogs service to access your work in the gogs repo.
- Reference [https://hub.docker.com/r/gogs/gogs](https://hub.docker.com/r/gogs/gogs) 
- Expect output: [https://hub.docker.com/repository/docker/your_dockerhub_id/gogs](https://hub.docker.com/repository/docker/your_dockerhub_id/gogs)

For Ericsson workforces:
- Use [ARMDocker](https://wiki.lmera.ericsson.se/wiki/ARM/Types/Docker_registry)
- Tag the container image using armdocker.rnd.ericsson.se/somepath/your_signum/gogs:v0.1 and push it to ARM docker ([armdocker.rnd.ericsson.se](https://armdocker.rnd.ericsson.se))
- Expected output: [arm.seli.gic.ericsson.se/ui/repos/tree/General/docker-v2-global-local/somepath/your_signum/gogs/v0.1](https://arm.seli.gic.ericsson.se/ui/repos/tree/General/docker-v2-global-local/somepath/your_signum/gogs/v0.1)

Expect that a single docker run command from consumer (assuming docker is available) can just bring up the gogs with all configurations/repo/files available.
```
docker run <parameters ... > <image:tag>
```

## **Task 99: Publish your work**

Please send the dockerhub repo or ARM docker repo link to the hiring manager or recruiter.
