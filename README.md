- [1. CTFd local deployment test](#1-ctfd-local-deployment-test)
  - [1.1. CTFd local K8s deployment](#11-ctfd-local-k8s-deployment)
  - [1.2. GitLab & Visual Studio Code](#12-gitlab--visual-studio-code)
    - [1.2.1. Gitlab code repository access](#121-gitlab-code-repository-access)
    - [1.2.2. Visual Studio Code setup](#122-visual-studio-code-setup)
  - [1.3. Install and setup Docker Desktop](#13-install-and-setup-docker-desktop)
  - [1.4. Install kubectl](#14-install-kubectl)
  - [1.5. Ingress NGINX controller](#15-ingress-nginx-controller)
  - [1.6. Cert-manager](#16-cert-manager)
  - [1.8. CTFd Kubernetes deployment](#18-ctfd-kubernetes-deployment)
  - [1.9. Backup & restore](#19-backup--restore)
- [2. Challenges](#2-challenges)
  - [2.1. Solve some challenges!](#21-solve-some-challenges)
  - [2.2. Make your own CTF challenge](#22-make-your-own-ctf-challenge)

# 1. CTFd local deployment test

CTFd is the Capture The Flag platform used by the Joint Cyber Range and in our case is running within Kubernetes. The original project's repository can be found [here](https://github.com/CTFd/CTFd), while [this](https://gitlab.com/hu-hc/jcr/platform/ctf-platform) is the location of the JCR's forked repository.

Clone this repository to get started with the Joint Cyber Range, change to its directory in your terminal and follow along.

**Contributing:** Do you find something that's wrong, got bad spelling, can be better explained, make more efficient use of words, could be extended or you just have general improvements? Please contribute to this and other Joint Cyber Range documentation.

## 1.1. CTFd local K8s deployment

- **Purpose:** to get started with using the Joint Cyber Range platform.

- **Preliminaries:** a Kubernetes environment for deployment, **Docker Desktop** is the easiest one to get started.

- **Note:** This documentation continues with Docker Desktop on Windows, while WSL2 will be utilized to run bash commands. The experience on MacOS and Linux should be close because of this.

## 1.2. GitLab & Visual Studio Code

The Joint Cyber Range utilizes the GitLab free tier for its code archive and CI pipeline. Whereas, VS Code is our IDE of choice. It is open source and offers a big catalogue of extensions. Git source control is natively integrated, for you to manage version control straight from within the IDE. Download and install it for [your system](https://code.visualstudio.com/download). Next you need to setup access to the code repository in GitLab.

### 1.2.1. Gitlab code repository access

You need a personal access token, to authenticate with the code repository using the HTTPS clone mechanism. For the personal access token, in case of GitLab, go to your account and click **Edit Profile**, then go to **Access Tokens**. Here you can type in your token name, an optional expiration date and select scopes. The scopes that need to be selected are **read_repository** and **write_repository**, if these are selected click **Create personal access token**. Save your new access token now, because you won't be able to access it again.

### 1.2.2. Visual Studio Code setup

Open a new (empty) VS Code Window and go to the Explorer (Ctrl+Shift+E). Click on Clone Respository and enter the repository’s URL: https://gitlab.com/hu-hc/jcr/jcr-getting-started. Login in to GitLab by using your personal access token. When the files are loaded, respond to the notice in the left corner and open the cloned repository.

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

<<<<<<< HEAD
## 1.6. Private container registry setup
=======
## 1.6. Cert-manager
Cert-manager is used to automatically renew certificates, that will be used to make HTTPS possible. A self-signed certificate is sufficient in your local setup. This is already provided in the form of a Kubernetes secret. You'll only have to install cert-manager as preparation:
```Bash
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.5.3/cert-manager.yaml
```

<!-- 
Create the initial self-signed certificate:
```Bash
openssl req -x509 -new -nodes -key ca.key -sha256 -subj "/CN=kubernetes.docker.internal" -days 1024 -out ca.crt
```

Use the self-signed certificate to create a Kubernetes secret: 
```Bash
kubectl create --save-config=true secret tls ca-key-pair --key=ca.key  --cert=ca.crt -n ctf-platform -o yaml > k8s/tls-ca-cert-sec
ret.yaml
```

Source: https://www.youtube.com/watch?v=JJTJfl-V_UM&list=WL&index=91
Talks about CA certificate is not a CA on MacOS. -->

<!-- Ad-hoc certificate without cert-manager:
Steps to reproduce a broken ad-hoc HTTPS setup after following the getting started repo
```
openssl req -x509 -newkey rsa:4096 -sha256 -nodes -keyout tls.key -out tls.crt -subj "/CN=kubernetes.docker.internal" -days 365
```
```
kubectl create --save-config=true secret tls kubernetes-docker-internal-tls --cert=tls.crt --key=tls.key -n ctf-platform -o yaml > k8s/tls-cert-secret.yaml
```
```
kubectl get secret kubernetes-docker-internal-tls -n ctf-platform
```
Add to the ingress resource under spec

```
tls:
  - hosts:
      - kubernetes.docker.internal
    secretName: kubernetes-docker-internal-tls
```
```
curl -k https:/kubernetes.docker.internal
curl --cacert tls.crt https:/kubernetes.docker.internal
curl --cacert tls.crt https:/kubernetes.docker.internal/setup
``` -->

## 1.7. Private container registry setup
>>>>>>> 97f0e758b3f3f6e20ffa6487a8d81cdbe25e5b77

For the CTFd container image you need to use a private registry. For local development speed. The objective is to get a secret stored in your Kubernetes cluster.

Create another personal access token in GitLab. This time select the following scopes: **read_registry** and **write_registry**.

You need to create a persistent Kubernetes secret. This will be used to authenticate to the private container registry and pull an image. But first, create a namespace for the application's resources.

```Bash
kubectl create namespace ctf-platform
```

Use the following command, with your own login credentials to create a Kubernetes secret and output it to Yaml format:

```bash
kubectl create --save-config=true secret docker-registry gitlab-pull --docker-server=registry.gitlab.com --docker-username={GitLab username} --docker-password={personal access token} --docker-email={email address} -n ctf-platform -o yaml > k8s/1-gitlab-pull-secret.yaml
```

<<<<<<< HEAD
The command has generated a Yaml manifest for you. Save this and don't share it, since it's only Base64 encoded.
=======
## 1.8. CTFd Kubernetes deployment
>>>>>>> 97f0e758b3f3f6e20ffa6487a8d81cdbe25e5b77

## 1.7. CTFd Kubernetes deployment

Use te following command to deploy all CTFd components described in the manifest:

```bash
kubectl apply -f k8s/
```

The service is now ready at [kubernetes.docker.internal](http://kubernetes.docker.internal).

Seeing results:

```bash
kubectl logs service/ctfd-service -n ctf-platform
```

You can now finish the setup. The required fields are: **Event Name** > **Admin Username**, **Admin Email** and **Admin Password**.

When you logout of the admin account, you're able to login with an account of a supported school (Hogeschool Utrecht).

If you want to cleanup, the following command can be used.

```bash
kubectl delete -f k8s/
```
<<<<<<< HEAD

## 1.8. Backup & restore

=======
## 1.9. Backup & restore
>>>>>>> 97f0e758b3f3f6e20ffa6487a8d81cdbe25e5b77
When you have the CTFd platform running and added some challenges or other things. Your're able to backup your CTF event to a zip file and restore it later if necessary.

You can restore a backup by importing the zip file included in this repository, it will load some basic challenges for you. To import the backup, go to **Admin Panel**, then click **Config** in the top menu. You will see **Backup** in the sidebar and click on **Import**. Choose the zip included or one of yourself and click on **Import**.
Admin account of the backup is: XXXXX

# 2. Challenges

CTF challenges basically run down to; IT/CYber Security puzzles, a digital scavenger hunt or escape room. Various types of CTF events exist, while CTFd makes it possible to host Jeopardy Style events.
Events focussed on professionals or internal training, try to simulate realistic Cyber Security scenarios. Attack/Defense games or Red vs Blue scenarios, is where the Joint Cyber Range will be heading to in the future.

## 2.1. Solve some challenges

In the zip file are some basic challenges included, from various CTF categories. Solve these to get a glimpse of how a CTF works.

Username: user
Password: user

**Note:** I wasn't able to deploy a container based challenge on CTFd. You can pull the image ```dockerburthet/ctf-ssh``` and try to solve the challenge.
**Title:** SSH
**Description:** Alice makes use of bad password hygiene on her server. Retrieve the content of the hidden message.

## 2.2. Make your own CTF challenge

Try to think of a basic CTF challenge that somebody else must be able to solve.

- Design and make a static text or upload based challenge first and save it in CTFd.
- Move on to creating a container-based challenge.

**Instructions container-based challenge:**

1. Create a dockerfile for the container(s) you want to include in the challenge. **Note:** All necessary files that it interacts with, must be included in the image.

2. Build and try your image. When you're satisfied, upload the container image to a container registry. 

3. Package it all up as a docker-compose file and test. If it works, you can encode your docker-compose file

4. Upload the challenge and test if the deployment and flag input work. Export your static and container-based challenges and share it with somebody to solve.
