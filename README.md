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
   - In host PowerShell:
      ```bash
      ssh-keygen -t ed25519
      ```
   - This creates:
      - Public key: ~/.ssh/id-ed25519
      - Private key: ~/.ssh/id_ed25519.pub
5. 
6. 

_What is host machine and guest machine?_


Use an ssh client in the host machine to ssh to the guest machine using public key authentication method.

_What is the IP of the guest machine? And is it accessible from the host machine? What configuration is required in order to have ssh access?_

_What are the difference between NAT and Bridge network?_

_How does the public key authentication method work? Will your private key travel to the remote server for authentication?_

After ssh via client terminal, upgrade the **Ubuntu system packages (packages pre-installed in the system)** to the latest ([https://ubuntu.com/server/docs/package-management](https://ubuntu.com/server/docs/package-management)).

Figure out what the **linux kernel version** is and then upgrade the kernel to the **latest LTS Enablement or Hardware Enablement (HWE) stack kernel ([https://ubuntu.com/kernel/lifecycle](https://ubuntu.com/kernel/lifecycle)). What is the kernel version after upgrade? 

## **Task 2: Install Docker Engine**

Install Docker Engine from Docker maintained repository (not Ubuntu maintained repository) [https://docs.docker.com/install/linux/docker-ce/ubuntu/](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

Run docker hello world to verify the installation.

_What are the relationships of the three components in Docker Engine, aka. docker-ce docker-ce-cli and containerd.io?_

## **Task 3: Install Gogs (a self-hosted git service)**

Install gogs from docker: [https://github.com/gogs/gogs/tree/main/docker](https://github.com/gogs/gogs/tree/main/docker) 

Choose the deployment configuration you prefer, just make sure it listens to port 3100 in the guest machine.

Expect output: Gogs is up and running at the guest machine, listening to port 3100. How to check if it is up and running as expected?

Open a browser in host machine and access gogs via [http://localhost:3100](http://localhost:3100)

_Is the port 3100 of the guest machine accessible from the host machine? If not, what configuration is required in order to expose it?_

For the first time, configure it following the installation steps 

*   Database uses SQLite3 (no external DB required)
*   Figure out the domain, ports and URL to be used
*   Create an admin user as well (username: root password: 123456)

## **Task 4: Create a Demo Organization/Repository in Gogs**

Sign in to gogs using admin account and create a demo org/repo named demo/go-web-hello-world (demo is organization name, go-web-hello-world is repository name)

Initiate the repo with a README.md and push the commit to master

Expected output: [http://localhost:3100/demo/go-web-hello-world](http://localhost:3100/demo/go-web-hello-world) 

## **Task 5: Install a Single Node Kubernetes Cluster Using Kind**

Follow instruction at [https://kind.sigs.k8s.io/docs/user/quick-start/](https://kind.sigs.k8s.io/docs/user/quick-start/).

Install kind first (use the stable binary rather than build from source to save time), then create a single node Kubernetes cluster using it.

Install kubectl client to interact with the cluster [https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/) 

Get cluster info, node info and list all pods in the cluster.

Check-in the kubeconfig file (~/.kube/config) into the gogs git repo. 

---

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

## **Task 7: Run the App in Container**

Build a docker image ($ docker build) for the web app, tag it go-app:latest ($ docker tag) and run it in a container ($ docker run), expose the service to 8082 (-p)

[https://docs.docker.com/engine/reference/commandline/build/](https://docs.docker.com/engine/reference/commandline/build/)

Check in the Dockerfile into git repo

Expect output (from guest machine):
```
curl http://localhost:8082
Go Web Hello World!
```
What is the size of the container image? Can it be optimized? If yes, how? Can you please make it less than 10MB? Check in the optimized Dockerfile into git repo.

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

## **Task 9: Install Kubernetes Dashboard**

Install Headlamp (https://headlamp.dev/) in the cluster using Yaml manifest method (not helm) and expose the service to nodeport 31081.

Can you access the dashboard URL via your host machine (not vm guest machine)? If not, how to configure the network to achieve it?

Open a browser in host machine and access the Dashboard via: [https://localhost:31081](https://localhost:31081) (asking for token).

Figure out how to generate a token to login to the dashboard.

Publish the procedure to the git repo.

---

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
