# Commands and Troubleshooting Cheat Sheet

Command reference for the Netflix Clone DevSecOps project: **Jenkins + Docker + SonarQube + Trivy + Helm + Kubernetes (EKS) + Argo CD**.

Commands are written for **Windows PowerShell** unless stated otherwise. Replace placeholders such as `<pod-name>`, image tags and the Helm chart directory (`./helm`) with your actual values.

## Table of Contents

1. [Git and GitHub](#1-git-and-github)
2. [Docker](#2-docker)
3. [Install Trivy inside the Jenkins container](#3-install-trivy-inside-the-jenkins-container)
4. [Trivy scanning and troubleshooting](#4-trivy-scanning-and-troubleshooting)
5. [Helm](#5-helm)
6. [AWS and EKS](#6-aws-and-eks)
7. [Kubernetes troubleshooting](#7-kubernetes-troubleshooting)
8. [NGINX Ingress Controller](#8-nginx-ingress-controller)
9. [Argo CD](#9-argo-cd)
10. [Jenkins troubleshooting](#10-jenkins-troubleshooting)
11. [SonarQube troubleshooting](#11-sonarqube-troubleshooting)
12. [Common problems and fixes](#12-common-problems-and-fixes)
13. [Cleanup](#13-cleanup)
14. [Quick revision](#14-quick-revision)
15. [Recommended troubleshooting sequence](#15-recommended-troubleshooting-sequence)

---

## 1. Git and GitHub

| Purpose | Command |
| --- | --- |
| Initialize Git | `git init` |
| Check status | `git status` |
| Add files | `git add .` |
| Commit changes | `git commit -m "Initial commit"` |
| Set branch | `git branch -M main` |
| Add remote | `git remote add origin https://github.com/vardhan-na/Netflix-clone.git` |
| Push code | `git push -u origin main` |
| Pull latest code | `git pull origin main` |
| View remotes | `git remote -v` |
| View commits | `git log --oneline` |

---

## 2. Docker

**Check Docker status**

```powershell
docker --version
docker info
docker ps
docker ps -a
```

**Images and containers**

```powershell
docker images
docker pull jenkins/jenkins:lts
docker pull sonarqube:lts-community
docker build -t netflix-clone .
docker run -d -p 8080:80 --name netflix-app netflix-clone
docker logs netflix-app
docker stop netflix-app
docker start netflix-app
docker rm -f netflix-app
docker rmi netflix-clone
```

**Access the Jenkins container**

```powershell
docker exec -it jenkins bash
docker exec -u root -it jenkins bash
docker logs jenkins
docker restart jenkins
```

**Resource usage**

```powershell
docker stats
docker system df
```

---

## 3. Install Trivy inside the Jenkins container

Run in PowerShell.

```powershell
docker ps
```

Install prerequisites:

```powershell
docker exec -u root jenkins bash -c "apt-get update && apt-get install -y wget gnupg"
```

Add the Trivy repository key:

```powershell
docker exec -u root jenkins bash -c "wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor -o /usr/share/keyrings/trivy.gpg"
```

Add the repository:

```powershell
docker exec -u root jenkins bash -c "echo 'deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb generic main' > /etc/apt/sources.list.d/trivy.list"
```

Install and verify:

```powershell
docker exec -u root jenkins bash -c "apt-get update && apt-get install -y trivy"
docker exec jenkins trivy --version
```

---

## 4. Trivy scanning and troubleshooting

**Filesystem scan**

```powershell
docker exec jenkins trivy fs .
```

**Scan a Docker image**

```powershell
docker exec jenkins trivy image vardhan2912/netflix-clone:34
```

**Show only HIGH and CRITICAL vulnerabilities**

```powershell
docker exec jenkins trivy image --severity HIGH,CRITICAL vardhan2912/netflix-clone:34
```

**JSON report**

```powershell
docker exec jenkins trivy image --format json -o trivy-report.json vardhan2912/netflix-clone:34
```

**HTML report** (uses Trivy's HTML template)

```powershell
docker exec jenkins trivy image --format template --template "@/contrib/html.tpl" -o trivy-report.html vardhan2912/netflix-clone:34
```

> Reports are written **inside the Jenkins container**. Copy one to your machine with:
> `docker cp jenkins:/trivy-report.html .` (adjust the path to the container's working directory if needed).

---

## 5. Helm

**Install on Windows and verify**

```powershell
winget install Helm.Helm
helm version
```

**Check the chart**

```powershell
helm lint ./helm
helm template netflix-app ./helm
```

**Install, upgrade, list, uninstall**

```powershell
helm install netflix-app ./helm
helm upgrade netflix-app ./helm
helm list -A
helm uninstall netflix-app
```

---

## 6. AWS and EKS

**Check installations**

```powershell
aws --version
eksctl version
kubectl version --client
```

**Configure AWS and check identity**

```powershell
aws configure
aws sts get-caller-identity
```

**Create the EKS cluster**

```powershell
eksctl create cluster --name Netflix-cluster --region us-west-2
```

**Update kubeconfig**

```powershell
aws eks update-kubeconfig --region us-west-2 --name Netflix-cluster
```

**Check the cluster**

```powershell
kubectl cluster-info
kubectl get nodes
kubectl get nodes -o wide
```

**Delete the cluster**

```powershell
eksctl delete cluster --name Netflix-cluster --region us-west-2
```

---

## 7. Kubernetes troubleshooting

Use these to check whether the Netflix application is running correctly in EKS.

**Cluster health**

```powershell
kubectl cluster-info
kubectl get nodes
kubectl get nodes -o wide
kubectl get namespaces
```

**Application resources**

```powershell
kubectl get pods
kubectl get deployments
kubectl get services
kubectl get ingress
kubectl get all
```

**Resources in every namespace**

```powershell
kubectl get pods -A
kubectl get deployments -A
kubectl get services -A
kubectl get ingress -A
```

**Netflix application**

```powershell
kubectl get pods -l app=netflix-app
kubectl get deployment netflix-app
kubectl get service netflix-app
kubectl describe deployment netflix-app
kubectl describe service netflix-app
```

**Pod details and logs**

```powershell
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl logs <pod-name> --previous
kubectl logs -f <pod-name>
```

**Events**

```powershell
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl get events -A --sort-by=.metadata.creationTimestamp
```

**Restart, rollout and scaling**

```powershell
kubectl rollout restart deployment/netflix-app
kubectl rollout status deployment/netflix-app
kubectl rollout history deployment/netflix-app
kubectl scale deployment netflix-app --replicas=2
```

**Check the container image in use**

```powershell
kubectl get pods -o wide
kubectl describe deployment netflix-app
```

---

## 8. NGINX Ingress Controller

**Install on AWS**

```powershell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.1/deploy/static/provider/aws/deploy.yaml
```

**Check the controller**

```powershell
kubectl get pods -n ingress-nginx
kubectl get services -n ingress-nginx
kubectl get deployments -n ingress-nginx
```

**Check the external LoadBalancer address and Ingress**

```powershell
kubectl get svc -n ingress-nginx
kubectl get ingress
kubectl describe ingress netflix-ingress
```

**Controller logs**

```powershell
kubectl logs -n ingress-nginx deployment/ingress-nginx-controller
```

The application hostname is `netflix.local`. Traffic is routed from the Ingress Controller to the Service and then to the application Pod.

---

## 9. Argo CD

**Install**

```powershell
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

**Check the installation**

```powershell
kubectl get pods -n argocd
kubectl get services -n argocd
kubectl get deployments -n argocd
```

**Expose the UI with a LoadBalancer** (PowerShell quoting)

```powershell
kubectl patch svc argocd-server -n argocd -p '{\"spec\": {\"type\": \"LoadBalancer\"}}'
```

**Get the external address**

```powershell
kubectl get svc argocd-server -n argocd
```

**Get the initial admin password** (username: `admin`)

```powershell
$ARGO_PASSWORD = kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}"
[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($ARGO_PASSWORD))
```

**Port-forward the UI** (then open `https://localhost:8081`)

```powershell
kubectl port-forward svc/argocd-server -n argocd 8081:443
```

**Argo CD CLI** (after installing the CLI and logging in)

```powershell
argocd version
argocd login <ARGOCD_SERVER>
argocd app list
argocd app get netflix-app
argocd app sync netflix-app
argocd app history netflix-app
argocd app get netflix-app --refresh
```

> The exact Argo CD application creation command / configuration (repo URL, path, sync policy): _TBD_

---

## 10. Jenkins troubleshooting

**Check the container**

```powershell
docker ps
docker ps -a
docker logs jenkins
docker logs --tail 100 jenkins
docker restart jenkins
```

**Enter the container**

```powershell
docker exec -it jenkins bash
```

**Check tools, workspace, disk and processes**

```powershell
docker exec jenkins java -version
docker exec jenkins trivy --version
docker exec -it jenkins bash -c "ls -la /var/jenkins_home/workspace/"
docker exec -it jenkins bash -c "ls -la /var/jenkins_home/workspace/Netflix-App"
docker exec jenkins df -h
docker exec jenkins ps aux
```

---

## 11. SonarQube troubleshooting

**Check the container**

```powershell
docker ps
docker logs sonarqube
docker logs --tail 100 sonarqube
```

**Check port 9000 and the web UI**

```powershell
Test-NetConnection localhost -Port 9000
```

Open `http://localhost:9000` in the browser. Analysis results and the Quality Gate status are shown in the SonarQube dashboard.

**Check connectivity from the Jenkins container**

```powershell
docker exec jenkins bash -c "curl -I http://sonarqube:9000"
```

> If Jenkins and SonarQube are not on the same Docker network, the name `sonarqube` may not resolve. Also, `localhost` inside the Jenkins container refers to the Jenkins container itself, not your Windows host.

---

## 12. Common problems and fixes

| Problem | Troubleshooting command |
| --- | --- |
| Docker not running | `docker info` |
| Jenkins not starting | `docker logs jenkins` |
| SonarQube not reachable | `Test-NetConnection localhost -Port 9000` |
| Trivy not installed | `docker exec jenkins trivy --version` |
| Kubernetes cluster not connected | `kubectl cluster-info` |
| Worker nodes not ready | `kubectl get nodes` |
| Pod stuck in Pending | `kubectl describe pod <pod-name>` |
| Pod in CrashLoopBackOff | `kubectl logs <pod-name> --previous` |
| ImagePullBackOff | `kubectl describe pod <pod-name>` |
| Service not working | `kubectl describe svc netflix-app` |
| Ingress not working | `kubectl describe ingress netflix-ingress` |
| Argo CD not running | `kubectl get pods -n argocd` |
| Deployment not updating | `kubectl rollout status deployment/netflix-app` |
| Helm deployment issue | `helm list -A` |
| Git push rejected | `git pull --rebase origin main` |

---

## 13. Cleanup

**Docker**

```powershell
docker stop netflix-app
docker rm netflix-app
docker rmi netflix-clone
docker system prune
```

**Kubernetes**

```powershell
kubectl delete deployment netflix-app
kubectl delete service netflix-app
kubectl delete ingress netflix-ingress
```

**Argo CD and NGINX Ingress Controller**

```powershell
kubectl delete namespace argocd
kubectl delete namespace ingress-nginx
```

**EKS cluster**

```powershell
eksctl delete cluster --name Netflix-cluster --region us-west-2
```

> **Important:** Deleting an EKS cluster does not necessarily remove every separately provisioned AWS resource (for example load balancers or volumes). Check your AWS console and billing after deletion.

---

## 14. Quick revision

The most useful commands for demos and interviews.

| Task | Command |
| --- | --- |
| Check Docker | `docker ps` |
| Check Jenkins | `docker logs jenkins` |
| Check Trivy | `trivy --version` |
| Check Helm | `helm version` |
| Check AWS | `aws sts get-caller-identity` |
| Check EKS nodes | `kubectl get nodes` |
| Check all Pods | `kubectl get pods -A` |
| Check deployments | `kubectl get deployments -A` |
| Check services | `kubectl get svc -A` |
| Check Ingress | `kubectl get ingress -A` |
| Check Argo CD | `kubectl get pods -n argocd` |
| Check app logs | `kubectl logs <pod-name>` |
| Check events | `kubectl get events -A` |
| Check Helm releases | `helm list -A` |
| Check rollout | `kubectl rollout status deployment/netflix-app` |

---

## 15. Recommended troubleshooting sequence

When the Netflix application is not accessible, run these in order:

```powershell
# 1. Check Docker
docker ps

# 2. Check EKS connection
kubectl cluster-info

# 3. Check worker nodes
kubectl get nodes

# 4. Check application Pods
kubectl get pods -A

# 5. Check application Deployment
kubectl get deployments

# 6. Check Services
kubectl get svc

# 7. Check Ingress
kubectl get ingress

# 8. Check Argo CD
kubectl get pods -n argocd

# 9. Check application logs
kubectl logs <pod-name>

# 10. Check Kubernetes events
kubectl get events -A --sort-by=.metadata.creationTimestamp
```

**Deployment flow**

```
GitHub (source code)
   ↓
Jenkins (CI/CD and security scans)
   ↓
Docker Hub (container image registry)
   ↓
GitHub Helm chart (updated image tag)
   ↓
Argo CD (GitOps synchronization)
   ↓
Amazon EKS (Deployment → Service → Ingress)
   ↓
Netflix Clone running
```
