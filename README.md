# 1. CTFd local deployment test

CTFd is the Capture The Flag platform used by the Joint Cyber Range and in our case is running within Kubernetes. The original project's repository can be found [here](https://github.com/CTFd/CTFd), while [this](https://gitlab.com/hu-hc/jcr/platform/ctf-platform) is the location of the JCR's forked repository.

Clone this repository to get started with the Joint Cyber Range, change to its directory in your terminal and follow along.

**Contributing:** Do you find something that's wrong, got bad spelling, can be better explained, make more efficient use of words, could be extended or you just have general improvements? Please contribute to this and other Joint Cyber Range documentation.

- [1. CTFd local deployment test](#1-ctfd-local-deployment-test)
  - [1.1. CTFd local K8s deployment](#11-ctfd-local-k8s-deployment)
  - [1.2. GitLab & Visual Studio Code](#12-gitlab--visual-studio-code)
    - [1.2.1. Gitlab code repository access](#121-gitlab-code-repository-access)
    - [1.2.2. Visual Studio Code](#122-visual-studio-code)
  - [1.3. Install and setup Docker Desktop](#13-install-and-setup-docker-desktop)
  - [1.4. Install kubectl](#14-install-kubectl)
  - [1.5. Ingress NGINX controller](#15-ingress-nginx-controller)
  - [1.6. Cert-manager](#16-cert-manager)
  - [1.7. CTFd Kubernetes deployment](#17-ctfd-kubernetes-deployment)
    - [1.7.1. Verification of deployed resources (optional)](#171-verification-of-deployed-resources-optional)
  - [1.8. Backup & restore](#18-backup--restore)
  - [1.9. Clean-up](#19-clean-up)

## 1.1. CTFd local K8s deployment

- **Purpose:** getting started with using the Joint Cyber Range platform and create a backup to restore.

- **Preliminaries:** a Kubernetes environment for deployment, **Docker Desktop** is the easiest one to get started with.

- **Note:** Most tools used are supported on Windows, MacOS and Linux, although this documentation continues using Docker Desktop on Windows, with  WSL2 as backend. Because of WSL and running Bash on Windows, the experience should be close on MacOS and Linux. The WSL distribution for Ubuntu 20.04 will be our Linux user environment and can be downloaded from the Microsoft Store.

## 1.2. GitLab & Visual Studio Code

The Joint Cyber Range utilizes the GitLab free tier for its code archive and CI pipeline. Whereas, VS Code is our IDE of choice. It is open source and offers a big catalogue of extensions. Git source control is natively integrated, for you to manage version control straight from within the IDE. Download and install it for [your system](https://code.visualstudio.com/download). Next you need to setup access to the code repository in GitLab.

### 1.2.1. Gitlab code repository access

You need a personal access token, to authenticate with the code repository using the HTTPS clone mechanism. For the personal access token, in case of GitLab, go to your account and click **Edit Profile**, then go to **Access Tokens**. Here you can type in your token name, an optional expiration date and select scopes. The scopes that need to be selected are **read_repository** and **write_repository**, if these are selected click **Create personal access token**. Save your new access token now, because you won't be able to access it again.

### 1.2.2. Visual Studio Code

Open a new (empty) VS Code Window and go to the Explorer (Ctrl+Shift+E). Click on Clone Respository and enter the repository’s URL: <https://gitlab.com/hu-hc/jcr/jcr-getting-started>. Login in to GitLab by using your personal access token. When the files are loaded, respond to the notice in the left corner and open the cloned repository.

## 1.3. Install and setup Docker Desktop

Download [Docker Desktop](https://www.docker.com/products/docker-desktop) for Windows or MacOS.

**Windows**: Docker Dekstop requires a WSL (recommended) backend, or Hyper-V on older Windows versions. You also need to have hardware virtualization enabled in your BIOS settings. Full instructions can be found [here](https://docs.docker.com/docker-for-windows/install). When installing Docker desktop, make sure on the configuration page **Install required Windows components for WSL 2** or the **Enable Hyper-V Windows Features** option is selected.

After Docker Desktop and WSL have been installed, choose a Linux distribution for WSL from the Windows Store (Ubuntu 20.04 is recommended). It's important to set the distribution to use WSL version 2, please follow [these instructions](https://docs.microsoft.com/en-us/windows/wsl/basic-commands#list-install-linux-distributions) for this. Now you can enable its integration with Docker Desktop, as described [here](https://docs.docker.com/desktop/windows/wsl/). This allows for running Docker commands within a WSL Terminal.

**MacOS**: On MacOS, there seems to be no additional requirements.

**Linux**: Docker desktop is not supported on Liunx. Use the Docker Engine instead with another local Kubernetes distribution, e.g. K3D, KIND or MiniKube.

Next, enable Kubernetes. Start Docker Desktop, go to **Settings** > **Kubernetes** and make sure **Enable Kubernetes** is checked.

## 1.4. Install kubectl

Kubectl is the CLI tool to interact with Kubernetes clusters. The following [link](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/) shows installation instructions for Linux, since we're working within our WSL terminal. Windows and MacOS instructions can be found in the menubar. Make sure your current directory is your home directory when installing, e.g. `cd ~/`.

Make sure your kube-config file is in the right location, and includes the correct cluster. On Windows it can be found in: `%USERPROFILE%\.kube\config` or (Mac/Linux `~/.kube/config`) .
For alternative locations you can make it known to kubectl with: e.g. `export KUBECONFIG=Path/kube.config` (Linux/MacOS version).

Test with: `kubectl get nodes`, you'll see your Kubernetes cluster nodes and name show up.

## 1.5. Ingress NGINX controller

The CTFd deployment on Kubernetes will make use of an ingress controller. That's a Kubernetes similar to reverse proxy and load balancing functionality. Install the Ingress Nginx controller for Docker Desktop.

```Bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.0.0/deploy/static/provider/cloud/deploy.yaml
```

Verify the installation.

```Bash
kubectl get pods -n ingress-nginx \
  -l app.kubernetes.io/name=ingress-nginx --watch
```

## 1.6. Cert-manager

Cert-manager is used to automatically renew certificates, that will be used to make HTTPS possible. A self-signed certificate is sufficient in your local setup. This is already provided in the form of a Kubernetes secret. You'll only have to install cert-manager as preparation.

```Bash
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.5.3/cert-manager.yaml
```

Verify the installation.

```Bash
kubectl get pods -n cert-manager --watch
```

**Manual (optional) certificate creation steps:**
Create the initial self-signed CA (Certificate Authority) certificate.

```Bash
openssl genrsa -out ca.key 2048
```

Create a certificate for the relevant DNS name with the newly created CA key.

```Bash
openssl req -x509 -new -nodes -key ca.key -sha256 -subj "/CN=kubernetes.docker.internal" -days 1024 -out ca.crt
```

Use the self-signed certificate to create a Kubernetes secret.

```Bash
kubectl create --save-config=true secret tls ca-key-pair --key=ca.key  --cert=ca.crt -n ctf-platform -o yaml > k8s/ca-key-pair-secret.yaml
```

Source: <https://www.youtube.com/watch?v=JJTJfl-V_UM&list=WL&index=91>
**Note:** Talks about CA certificate is not a CA on MacOS, but is from 2018.

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

For the CTFd container image you need to use a private registry. For local development speed. The objective is to get a secret stored in your Kubernetes cluster.

Create another personal access token in GitLab. This time select the following scopes. **read_registry** and **write_registry**.

You need to create a persistent Kubernetes secret. This will be used to authenticate to the private container registry and pull an image. But first, create a namespace for the application's resources.

```Bash
kubectl create namespace ctf-platform
```

Use the following command, with your own login credentials to create a Kubernetes secret and output it to Yaml format.

```bash
kubectl create --save-config=true secret docker-registry gitlab-pull --docker-server=registry.gitlab.com --docker-username={GitLab username} --docker-password={personal access token} --docker-email={email address} -n ctf-platform -o yaml > k8s/1-gitlab-pull-secret.yaml
```

The command has generated a Yaml manifest for you. Save this and don't share it, since it's only Base64 encoded.

## 1.7. CTFd Kubernetes deployment

Use te following command to deploy all CTFd components described in the manifest.

```bash
kubectl apply -f k8s/
```

The output should be.

```Bash
secret/gitlab-pull created
service/ctfd-service created
deployment.apps/ctfd created
issuer.cert-manager.io/ca-issuer created
certificate.cert-manager.io/kubernetes-docker-internal created
ingress.networking.k8s.io/ctfd-ingress created
secret/ca-key-pair created
```

Seeing results of the application (CTFd) component.

```bash
kubectl logs service/ctfd-service -n ctf-platform
```

The service is now ready at [kubernetes.docker.internal](http://kubernetes.docker.internal).

You can now finish the setup. The required fields are: **Event Name** > **Admin Username**, **Admin Email** and **Admin Password**.

When you logout of the admin account, you're able to login with an account of a supported school (Hogeschool Utrecht).

### 1.7.1. Verification of deployed resources (optional)

Information about the application's deployment can be displayed with.

```Bash
kubectl describe deployment ctfd -n ctf-platform
```

Service information can be displayed with.

```Bash
kubectl describe service -n ctf-platform
```

Check the created Kubernetes secrets.

```Bash
kubectl get secret ca-key-pair kubernetes-docker-internal-tls gitlab-pull -n ctf-platform
kubectl describe secret ca-key-pair kubernetes-docker-internal-tls gitlab-pull -n ctf-platform
```

Check that the ingress resource `HOSTS` is mapped to [kubernetes.docker.internal](http://kubernetes.docker.internal), with `ADDRESS` mapped to [localhost](http://localhost).

```Bash
kubectl get ingress -n ctf-platform
kubectl describe ingress -n ctf-platform
```

Check the custom resources related to cert-manager.

```Bash
kubectl get Issuers,Certificates --all-namespaces
```

Or see all the results at once.

```Bash
kubectl get namespace,deployments,service,ingress,secret,Issuer,Certificates -n ctf-platform
kubectl describe deployments,service,ingress,secret,Issuer,Certificates -n ctf-platform
```

## 1.8. Backup & restore

When you have the CTFd platform running and added some challenges or other things. Your're able to backup your CTF event to a zip file and restore it later if necessary.

You can restore a backup by importing the zip file included in this repository, it will load some basic challenges for you. To import the backup, go to **Admin Panel**, then click **Config** in the top menu. You will see **Backup** in the sidebar and click on **Import**. Choose the zip included or one of yourself and click on **Import**.

Admin credentials:
Username: admin
Password: jcr

## 1.9. Clean-up

The following command will delete all custom created resources.

```Bash
kubectl delete -f k8s/
```

The namespace you've created, will have to be manually deleted.

```Bash
kubectl delete namespace ctf-platform
```

Uninstall cert-manager.

```Bash
kubectl delete -f https://github.com/jetstack/cert-manager/releases/download/v1.5.3/cert-manager.yaml
```

Uninstall the NGINX ingress controller.

```Bash
kubectl delete -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.0.0/deploy/static/provider/cloud/deploy.yaml
```

If you really want to be sure all resources are deleted or when you run into trouble, then the cluster can always be resetted.
