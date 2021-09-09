- [1. CTFd local deployment](#1-ctfd-local-deployment)
  - [1.1. CTFd local K8s deployment](#11-ctfd-local-k8s-deployment)
  - [1.2. Visual Studio Code](#12-visual-studio-code)
  - [1.3. Install and setup Docker Desktop](#13-install-and-setup-docker-desktop)
  - [1.4. Install kubectl](#14-install-kubectl)
  - [1.5. Ingress NGINX controller](#15-ingress-nginx-controller)
  - [1.6. Set up a private container registry](#16-set-up-a-private-container-registry)
  - [1.7. CTFd Kubernetes deployment](#17-ctfd-kubernetes-deployment)
  - [1.8. Backup & restore](#18-backup--restore)
- [2. Challenges](#2-challenges)
  - [2.1. Solve some challenges!](#21-solve-some-challenges)
  - [2.2. Make your own CTF challenge](#22-make-your-own-ctf-challenge)

# 1. CTFd local deployment

CTFd is the Capture The Flag platform used by the Joint Cyber Range and in our case is running within Kubernetes. The original project's repository can be found [here](https://github.com/CTFd/CTFd), while [this](https://gitlab.com/hu-hc/jcr/platform/ctf-platform) is the location of the JCR's forked repository.

Clone this repository to get started with the Joint Cyber Range, change to its directory in your terminal and follow along. 

**Contributing:** Do you find something that's wrong, got bad spelling, can be better explained, make more efficient use of words, could be extended or you just have general improvements? Please contribute to this and other Joint Cyber Range documentation. 

## 1.1. CTFd local K8s deployment

- **Purpose:** to get started with using the Joint Cyber Range platform.

- **Preliminaries:** a Kubernetes environment for deployment, **Docker Desktop** is the easiest one to get started.

- **Note:** This documentation continues with Docker Desktop on Windows, while WSL2 will be utilized to run bash commands. The experience on MacOS and Linux should be close because of this.

## 1.2. Visual Studio Code

VS Code is our IDE of choice. It is open source and offers a big catalogue of extensions. Git source control is natively integrated, for you to manage version control straight from within the IDE. Download and install it for [your system](https://code.visualstudio.com/download). Open a new (empty) VS Code Window and go to the Explorer (Ctrl+Shift+E). Click on Clone Respository and enter the repository’s URL: https://gitlab.com/hu-hc/jcr/jcr-getting-started. 

## 1.3. Install and setup Docker Desktop

Download [Docker Desktop](https://www.docker.com/products/docker-desktop) for Windows or MacOS.

- **Windows**: Docker Dekstop requires a WSL (recommended) backend, or Hyper-V on older Windows versions. You also need to have hardware virtualization enabled in your BIOS settings. Full instructions can be found [here](https://docs.docker.com/docker-for-windows/install). When installing Docker desktop, make sure on the configuration page **Install required Windows components for WSL 2** or the **Enable Hyper-V Windows Features** option is selected.

  **Optional:** After the Docker Desktop install you can follow the instructions from [step 5](https://docs.microsoft.com/en-us/windows/wsl/install-win10#step-6---install-your-linux-distribution-of-choice), to install a WSL distribution and enable integration with Docker Desktop. This allows for running Docker commands within a WSL Terminal.

- **MacOS**: On MacOS, there seems to be no additional requirements.

- **Linux**: Docker desktop is not supported on Liunx. Use the Docker Engine instead with another local Kubernetes distribution, e.g. K3D, KIND or MiniKube.

Next, enable Kubernetes. Start Docker Desktop, go to **Settings** > **Kubernetes** and make sure **Enable Kubernetes** is checked.

## 1.4. Install kubectl

Kubectl is the CLI tool to interact with Kubernetes clusters. The following [link](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/) shows installation instructions for Linux, since we're working within our WSL terminal. Windows and MacOS instructions can be found in the menubar.

Make sure your kube-config file is in the right location, and includes the correct cluster. On Windows it can be found in: `%USERPROFILE%\.kube\config` or (Mac/Linux `~/.kube/config`) .
For alternative locations you can make it known to kubectl with: e.g. `export KUBECONFIG=Path/kube.config` (Linux/MacOS version).

Test with: `kubectl get nodes`, you'll see your Kubernetes cluster name show up.

## 1.5. Ingress NGINX controller

The CTFd deployment on Kubernetes will make use of an ingress controller. That's a Kubernetes similar to reverse proxy and load balancing functionality. Install the Ingress Nginx controller for Docker Desktop:
```Bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.0.0/deploy/static/provider/cloud/deploy.yaml
```
## 1.6. Set up a private container registry

For the CTFd container image you need to use a private registry. For local development speed. The objective is to get a secret stored in your Kubernetes cluster.

To do this you first need a personal access token to authenticate with the registry. For the personal access token, in case of GitLab, go to your account and click **Edit Profile**, then go to **Access Tokens**. Here you can type in your token name, an optional expiration date and select scopes. The scopes that need to be selected are **read_registry** and **write_registry**, if these are selected click **Create personal access token**. Save your new access token now, because you won't be able to access it again.

We need to create a persistent Kubernetes secret. This will be used to authenticate to the private container registry and pull an image. Use the following command, with your own login credentials:

```bash
kubectl create secret docker-registry gitlab-pull --docker-server=registry.gitlab.com --docker-username={GitLab username} --docker-password={personal access token} --docker-email={email address} -n ctf-platform -o yaml > k8s/2-gitlab-pull-secret.yaml
```
The command has generated a Yaml manifest for you. Save this and don't share it, since it's only Base64 encoded. 
## 1.7. CTFd Kubernetes deployment


Use te following command to deploy all CTFd components described in the manifest:

```bash
kubectl apply -f k8s/
```

The service is now ready at [kubernetes.docker.internal](http://kubernetes.docker.internal). 

Seeing results:
```bash
kubectl logs service/ctfd-service
```


You can now finish the setup. The required fields are: **Event Name** > **Admin Username**, **Admin Email** and **Admin Password**.

If you want to cleanup, the following command can be used.
```bash
kubectl delete -f k8s/
```
## 1.8. Backup & restore
When you have the CTFd platform running and added some challenges or other things. Your're able to backup your CTF event to a zip file and restore it later if necessary.

You can restore a backup by importing the zip file included in this repository, it will load some basic challenges for you. To import the backup, go to **Admin Panel**, then click **Config** in the top menu. You will see **Backup** in the sidebar and click on **Import**. Choose the zip included or one of yourself and click on **Import**. 

# 2. Challenges
CTF challenges basically run down to; IT/CYber Security puzzles, a digital scavenger hunt or escape room. Various types of CTF events exist, while CTFd makes it possible to host Jeopardy Style events. 
Events focussed on professionals or internal training, try to simulate realistic Cyber Security scenarios. Attack/Defense games or Red vs Blue scenarios, is where the Joint Cyber Range will be heading to in the future.

## 2.1. Solve some challenges!
In the zip file are some basic challenges included, from various CTF categories. Solve these to get a glimpse of how a CTF works. 

Username: user
Password: user

**Note:** I wasn't able to deploy a container based challenge on CTFd. You can pull the image```dockerburthet/ctf-ssh``` and try to solve the challenge.
**Title:** SSH
**Description:** Alice makes use of bad password hygiene on her server. Retrieve the content of the hidden message.
## 2.2. Make your own CTF challenge
Try to think of a basic CTF challenge that somebody else must be able to solve. 
- Design and make a static text or upload based challenge first and save it in CTFd.
- Move on to creating a container-based challenge. 

**Instructions container-based challenge:** 
**1.** Create a dockerfile for the container(s) you want to include in the challenge. **Note:** All necessary files that it interacts with, must be included in the image.

**2.** Build and try your image. When you're satisfied, upload the container image to a container registry. 

**3.** Package it all up as a docker-compose file and test. If it works, you can encode your docker-compose file

**4.** Upload the challenge and test if the deployment and flag input work. Export your static and container-based challenges and share it with somebody to solve.