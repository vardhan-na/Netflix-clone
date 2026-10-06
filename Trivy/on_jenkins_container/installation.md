## Install Trivy on jenkins container(run cmd on powershell)

## Find Jenkins container
```bash
docker ps
```
## Update Packages & Install Required Toolst

```bash
docker exec -u root jenkins bash -c "apt-get update && apt-get install -y wget gnupg"
```
## Download & Install the Trivy Repository GPG Key

```bash
docker exec -u root jenkins bash -c "wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor -o /usr/share/keyrings/trivy.gpg"
```
## Add the Trivy APT Repository

```bash
docker exec -u root jenkins bash -c "echo 'deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb generic main' > /etc/apt/sources.list.d/trivy.list"
```

## Install Trivy

```bash
docker exec -u root jenkins bash -c "apt-get update && apt-get install -y trivy"
```

## Verify Trivy Installation

```bash
docker exec jenkins trivy --version
```
