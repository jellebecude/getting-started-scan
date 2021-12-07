# 1. CTFd local deployment

CTFd is the Capture The Flag platform used by the Joint Cyber Range and in our case is running within Kubernetes. The original project's repository can be found [here](https://github.com/CTFd/CTFd), while [this](https://gitlab.com/hu-hc/jcr/platform/ctf-platform) is the location of the JCR's forked repository.

- **Purpose:** getting started with using the Joint Cyber Range platform and create a backup to restore.

- **Preliminaries:** a Kubernetes environment for deployment, **Docker Desktop** is the easiest one to get started with.

- **Note:** Most tools used are supported on Windows, MacOS and Linux, although this documentation continues using Docker Desktop on Windows, with  WSL2 as backend. Because of WSL and running Bash on Windows, the experience should be close on MacOS and Linux. The WSL distribution for Ubuntu 20.04 will be our Linux user environment and can be downloaded from the Microsoft Store.

**Contributing:** Do you find something that's wrong, got bad spelling, can be better explained, make more efficient use of words, could be extended or you just have general improvements? Please contribute to this guide and other Joint Cyber Range documentation. You can make a [new issue](https://gitlab.com/hu-hc/jcr/getting-started/-/issues/new?issue%5Bmilestone_id%5D=) on the GitLab repo.

You could also take matters in your own hands by creating a [new branch](https://gitlab.com/hu-hc/jcr/getting-started/-/branches/new), making your changes, doing a commit and perform a `Pull request` or in GitLab's terms, creating a new [Merge request](https://gitlab.com/hu-hc/jcr/getting-started/-/merge_requests/new). Your changes will be reviewed by a Joint Cyber Range associated developer.

- [1. CTFd local deployment](#1-ctfd-local-deployment)
  - [1.1. GitLab and Visual Studio Code](#11-gitlab-and-visual-studio-code)
    - [1.1.1. Gitlab code repository access](#111-gitlab-code-repository-access)
    - [1.1.2. Visual Studio Code](#112-visual-studio-code)
  - [1.2. Install and setup Docker Desktop](#12-install-and-setup-docker-desktop)
    - [1.2.1. Resetting Docker Desktop and Kubernetes](#121-resetting-docker-desktop-and-kubernetes)
  - [1.3. Install kubectl](#13-install-kubectl)
  - [1.4. Ingress NGINX controller](#14-ingress-nginx-controller)
  - [1.5. Kubernetes namespace and choice for data persistence](#15-kubernetes-namespace-and-choice-for-data-persistence)
  - [1.6. CTFd Kubernetes deployment](#16-ctfd-kubernetes-deployment)
    - [1.6.1. Verify the resources](#161-verify-the-resources)
    - [1.6.2. Verification of deployed resources (optional)](#162-verification-of-deployed-resources-optional)
  - [1.7. Backup & restore](#17-backup--restore)
  - [1.8. Clean-up](#18-clean-up)
    - [1.8.1. Ephemeral storage](#181-ephemeral-storage)
    - [1.8.2. Persistent storage](#182-persistent-storage)
    - [1.8.3. Delete the namespace and Ingress controller](#183-delete-the-namespace-and-ingress-controller)
  - [1.9. Known Issues](#19-known-issues)
    - [1.9.1. CTFd not accessible due to port conflict](#191-ctfd-not-accessible-due-to-port-conflict)
    - [1.9.2. CTFd logs in Docker Desktop](#192-ctfd-logs-in-docker-desktop)
    - [1.9.3. NGINX ingress external IP pending](#193-nginx-ingress-external-ip-pending)
  - [1.10. Next steps](#110-next-steps)

## 1.1. GitLab and Visual Studio Code

The Joint Cyber Range utilizes the GitLab free tier for its code archive and CI pipeline. Whereas, VS Code is our IDE of choice. It is open source and offers a big catalogue of extensions. Git source control is natively integrated, for you to manage version control straight from within the IDE. Download and install it for [your system](https://code.visualstudio.com/download). Next you need to setup access to the code repository in GitLab.

### 1.1.1. Gitlab code repository access

You need a personal access token, to authenticate with the code repository using the HTTPS clone mechanism. For the personal access token, in case of GitLab, go to [Access Tokens under your GitLab profile preferences](https://gitlab.com/-/profile/personal_access_tokens). Here you can type in your token name, an optional expiration date and select scopes. The scopes that need to be selected are **read_repository** and **write_repository**, if these are selected click **Create personal access token**. Save your new access token now, because you won't be able to access it again.

Besides an Access Token you also need to import a SSH key into GitLab. If you don't already have one to import, create a SSH key pair by accepting the defaults on the command `ssh-keygen` and see the results with `cat`.

```Bash
ssh-keygen
cat ~/.ssh/id_rsa.pub
```

Go the [SSH Keys under GitLab profile preferences](https://gitlab.com/-/profile/keys) and import the public key.

### 1.1.2. Visual Studio Code

Open a new (empty) VS Code Window and go to the Explorer (Ctrl+Shift+E). Click on Clone Respository and enter the repository’s URL: <https://gitlab.com/hu-hc/jcr/jcr-getting-started>. Login in to GitLab by using your personal access token. When the files are loaded, respond to the notice in the right corner and open the cloned repository.

## 1.2. Install and setup Docker Desktop

Download [Docker Desktop](https://www.docker.com/products/docker-desktop) for Windows or MacOS.

**Windows**: Docker Dekstop requires a WSL (recommended) backend, or Hyper-V on older Windows versions. You also need to have hardware virtualization enabled in your BIOS settings. Full instructions can be found [here](https://docs.docker.com/docker-for-windows/install). When installing Docker desktop, make sure on the configuration page **Install required Windows components for WSL 2** or the **Enable Hyper-V Windows Features** option is selected.

After Docker Desktop and WSL have been installed, choose a Linux distribution for WSL from the Windows Store (Ubuntu 20.04 is recommended). It's important to set the distribution to use WSL version 2, please follow [these instructions](https://docs.microsoft.com/en-us/windows/wsl/basic-commands#list-install-linux-distributions) for this. Now you can enable its integration with Docker Desktop, as described [here](https://docs.docker.com/desktop/windows/wsl/). This allows for running Docker commands within a WSL Terminal.

**MacOS**: On MacOS, there seems to be no additional requirements.

**Linux**: Docker desktop is not supported on Linux. Use the Docker Engine instead with another local Kubernetes distribution, e.g. K3D, KIND or MiniKube.

Next, enable Kubernetes. Start Docker Desktop, go to **Settings** > **Kubernetes** and make sure **Enable Kubernetes** is checked. 

### 1.2.1. Resetting Docker Desktop and Kubernetes

You can reset the Kubernetes cluster and delete all created resources by going to **Settings** > **Kubernetes**, then click `Reset Kubernetes Cluster`. All reset functions are also conveniently listed when you click on the `Bug` icon, including `Clean / Purge data` and `Reset to factory defaults`. When you do this because you've ran into a problem and are unable to fix it, sometimes it's required to completely re-install Docker desktop.

## 1.3. Install kubectl

Kubectl is the CLI tool to interact with Kubernetes clusters. The following [link](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/) shows installation instructions for Linux, since we're working within our WSL terminal. Windows and MacOS instructions can be found in the menubar. Make sure your current directory is your home directory when installing, e.g. `cd ~/`.

Make sure your kube-config file is in the right location, and includes the correct cluster. On Windows it can be found in: `%USERPROFILE%\.kube\config` or (Mac/Linux `~/.kube/config`) .
For alternative locations you can make it known to kubectl with: e.g. `export KUBECONFIG=PathTo/kube.config` (Linux/MacOS version).

Test with: `kubectl get nodes`, you'll see your Kubernetes cluster nodes and name show up.

## 1.4. Ingress NGINX controller

The CTFd deployment on Kubernetes will make use of an ingress controller. That's a Kubernetes similar to reverse proxy and load balancing functionality. Install the Ingress Nginx controller for Docker Desktop.

```Bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.0.0/deploy/static/provider/cloud/deploy.yaml
```

Verify the installation.

```Bash
kubectl get pods -n ingress-nginx \
  -l app.kubernetes.io/name=ingress-nginx
```

Expected output:
```Bash
NAME                                       READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-b42k6       0/1     Completed   0          46s
ingress-nginx-admission-patch-6zpd5        0/1     Completed   1          46s
ingress-nginx-controller-fd7bb8d66-qltbg   0/1     Running     0          46s
ingress-nginx-controller-fd7bb8d66-qltbg   1/1     Running     0          50s
```

**Tip:** you can continuously watch the resource by adding the `--watch` paramater. 

## 1.5. Kubernetes namespace and choice for data persistence

Before you create any resources, you'll have to create the namespace they will reside in. The app can run with or without data persistence. `dev-minimal` will run the app without persistence, whereas `dev-data` wil run the app with data persistence.

```Bash
NAMESPACE=dev-minimal
kubectl create namespace $NAMESPACE
```

## 1.6. CTFd Kubernetes deployment

Use the following command to deploy all CTFd components.

```bash
kubectl apply -k kustomize/$NAMESPACE
```

The output should be similar to.

```Bash
namespace/dev-minimal configured
serviceaccount/dev-minimal-ctfd created
clusterrole.rbac.authorization.k8s.io/dev-minimal-ctfd created
clusterrolebinding.rbac.authorization.k8s.io/dev-minimal-ctfd-kube created
configmap/dev-minimal-configmap-ctfd-h2ht7cd778 created
secret/dev-minimal-selfsigned-tls-secret created
service/dev-minimal-ctfd created
deployment.apps/dev-minimal-ctfd created
ingress.networking.k8s.io/dev-minimal-ingress created
```

### 1.6.1. Verify the resources

See in what state the pod is. With the status `ImagePullBackOff` your provided GitLab credentials in the previously created `gitlab-pull`  secret are insufficient. A second possibility is that the GitLab access permissions on the repository for your account are insufficient.

```Bash
kubectl get pods -n $NAMESPACE
```

The output should be as follows.

```Bash
NAME                    READY   STATUS    RESTARTS   AGE
ctfd-78f786956f-wvbkt   1/1     Running   0          7m9s
```

**Tip:** you can continuously watch the resource by adding the `--watch` paramater. 

Seeing results of the application (CTFd) component.

```bash
kubectl logs service/ctfd-service -n $NAMESPACE
```

Expected output.

```Bash
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running stamp_revision  -> 75e8ab9a0014
 * Loaded module, <module 'CTFd.plugins.CTFd-oauth-openid' from '/opt/CTFd/CTFd/plugins/CTFd-oauth-openid/__init__.py'>
 * Loaded module, <module 'CTFd.plugins.challenges' from '/opt/CTFd/CTFd/plugins/challenges/__init__.py'>
 * Loaded module, <module 'CTFd.plugins.challengesuser' from '/opt/CTFd/CTFd/plugins/challengesuser/__init__.py'>
 * Loaded module, <module 'CTFd.plugins.dynamic_challenges' from '/opt/CTFd/CTFd/plugins/dynamic_challenges/__init__.py'>
 * Loaded module, <module 'CTFd.plugins.extra-roles' from '/opt/CTFd/CTFd/plugins/extra-roles/__init__.py'>
 * Loaded module, <module 'CTFd.plugins.flags' from '/opt/CTFd/CTFd/plugins/flags/__init__.py'>
 * Loaded module, <module 'CTFd.plugins.kubernetes' from '/opt/CTFd/CTFd/plugins/kubernetes/__init__.py'>
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
Starting CTFd
[2021-10-22 14:33:32 +0000] [1] [INFO] Starting gunicorn 20.0.4
[2021-10-22 14:33:32 +0000] [1] [INFO] Listening at: http://0.0.0.0:8000 (1)
[2021-10-22 14:33:32 +0000] [1] [INFO] Using worker: gevent
[2021-10-22 14:33:32 +0000] [13] [INFO] Booting worker with pid: 13
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
```

The service is now ready at [kubernetes.docker.internal](http://kubernetes.docker.internal). Provided you have no other services running on port 443. Expect a warning about an invalid certificate. Depending on your browser, you may need to go into incognito mode to proceed. 

You can now finish the setup. The required fields are: **Event Name** > **Admin Username**, **Admin Email** and **Admin Password**.

When you logout of the admin account, you're able to login with other accounts. Ones that you manually created as admin or affiliated / supported educational institutions by the SURFconext federated identity. 

### 1.6.2. Verification of deployed resources (optional)

Information about the application's deployment can be displayed with.

```Bash
kubectl describe deployment ctfd -n $NAMESPACE
```

Service information can be displayed with.

```Bash
kubectl describe service -n $NAMESPACE
```

Check the created Kubernetes secrets.

```Bash
kubectl get secret kubernetes-docker-internal-tls gitlab-pull -n $NAMESPACE
kubectl describe secret kubernetes-docker-internal-tls gitlab-pull -n $NAMESPACE
```

Check that the ingress resource `HOSTS` is mapped to [kubernetes.docker.internal](http://kubernetes.docker.internal), with `ADDRESS` mapped to [localhost](http://localhost).

```Bash
kubectl get ingress -n $NAMESPACE
kubectl describe ingress -n $NAMESPACE
```

Or see all the results at once.

```Bash
kubectl get namespace,deployments,service,ingress,secret -n $NAMESPACE
kubectl describe deployments,service,ingress,secret -n $NAMESPACE
```

## 1.7. Backup & restore

When you have the CTFd platform running and added some challenges or other things. Your're able to backup your CTF event to a zip file and restore it later if necessary.

You can restore a backup by importing the zip file included in this repository, it will load some basic challenges for you. To import the backup, go to **Admin Panel**, then click **Config** in the top menu. You will see **Backup** in the sidebar and click on **Import**. Choose the zip included or one of yourself and click on **Import**. When uploading seems to get stuck, reload the site by browsing to the homepage. You can login with the credentials stored in the backup of the CTF event.

**Admin credentials:**
Username: _admin_
Password: _jcr_

## 1.8. Clean-up

### 1.8.1. Ephemeral storage

The following command will delete all custom created resources.

```Bash
kubectl delete -f k8s/
```

### 1.8.2. Persistent storage

Leave Persistent Volumes and its data. 

```Bash
kubectl delete -f k8s-with-persistence/
```

Delete all resources, including Persistent Volumes 

```Bash
kubectl delete -f k8s-with-persistence/ -R
```

### 1.8.3. Delete the namespace and Ingress controller

The namespace you've created, will have to be manually deleted.

```Bash
kubectl delete namespace ctf-platform
```

Uninstall the NGINX ingress controller.

```Bash
kubectl delete -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.0.0/deploy/static/provider/cloud/deploy.yaml
```

If you really want to be sure all resources are deleted or when you run into trouble, then the cluster can always be resetted. Instructions at [1.3.1. Resetting Docker Desktop and Kubernetes](README.md#131-resetting-docker-desktop-and-kubernetes).

## 1.9. Known Issues



### 1.9.1. CTFd not accessible due to port conflict

When deploying the CTFd platform locally there is a possibility that there is a conflict with other running applications on the host system. The CTFd platform listens on the port 443. When on Windows you can use the `Get-NetTCPConnection` command to see if there are other processes that run on port 443.

Open Powershell and run this command:
```Powershell
try {Get-NetTCPConnection -ea stop -LocalPort 443 | select local*,state,@{Name="Process";Expression={(Get-Process -Id $_.OwningProcess).ProcessName}}} catch {write-output "Port 443 not in use"}
```
You now see a list of processes that run on the 443 port or a message showing that port 443 is not in use.
If there is more than 1 process that runs on the 443 port there is a possibility that the other processes are hijacking the http request to the CTFd website. 

To resolve this issue you can close the other processes.

VMware workstation is one application that can conflict with the CTFd platform. It has a deprecated feature for sharing VMs that defaults to port 443. To resolve this you can turn the feature off.

### 1.9.2. CTFd logs in Docker Desktop

CTFd logs can be viewed with the provided command.

```Bash
kubectl logs service/ctfd-service -n ctf-platform
```

Docker desktop also provides logs for each container, though the messages may arrive in a different order.

### 1.9.3. NGINX ingress external IP pending

If CTFd is not accessible, check if the Service of the NGINX ingress has an external IP entry for `localhost`. Command to check the status of the Ingress. 

```Bash
kubectl get services -n ingress-nginx
```

Desired output. 

```Bash
NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.102.14.41     localhost     80:30214/TCP,443:31156/TCP   19h
ingress-nginx-controller-admission   ClusterIP      10.101.177.213   <none>        443/TCP                      19h
```

Actual output. 

```Bash
NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.102.14.41     <pending>     80:30214/TCP,443:31156/TCP   19h
ingress-nginx-controller-admission   ClusterIP      10.101.177.213   <none>        443/TCP                      19h
```

Fix: If it is pending, remove Docker Desktop, remove all the folders listed below and reinstall. 

```
%USERPROFILE%\AppData\Local\Docker
%USERPROFILE%\AppData\Roaming\Docker
%ProgramData%\Docker
%ProgramFiles%\Docker
```

## 1.10. Next steps

- [Create static & container-based challenges](/challenges/README.md)
- [JCR development & contributing](https://gitlab.com/hu-hc/jcr/platform/ctf-platform/-/blob/master/README.md)