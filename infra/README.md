# DevOps Pipeline Setup

Setup guide for Spring Petclinic DevOps pipeline with Jenkins, SonarQube, OWASP ZAP, Prometheus, and Grafana. The production and staging VMs are setup using Vagrant and deployed to via Ansible.


## Team - 3 Members
1. Karthik Chakkalakal (kchakkal@andrew.cmu.edu)
2. Sarvesh Navare (snavare@andrew.cmu.edu)
3. Srushti Venkatesh Reddy (srushtiv@cs.cmu.edu)

## Prerequisites

### Host Specifications
The majority of testing and implementation was done a host with below specifications

- CPU/ Chip - Apple M4 (ARM-based)
- Memory - 16 GB
- OS - MacOS Tahoe 26.1

Note: We have tried to overcome the architecture issues that can occur with running VMs having different architecture than the host.
As such, the Vagrant box being used during setup (Vagrantfile) contains Ubuntu 24.04 for both x64 and ARM and should automatically pick the correct box based on the underlying host architecture. 
This same implementation was tested on a host with Windows 11 (x64 based) architecture, achieving proper implementation without requiring any changes to code, but enough testing has not been done to ensure a fully bug free implementation in this case.

### Required Software on Host Machine
- **Docker**
- **VirtualBox**
- **Vagrant**
- **Git**
- **curl** (for API calls)
- **jq** (for parsing JSON)
 
### Staging and Production VM Specifications (via Vagrant)
- CPU - 1 core
- Memory - 1024 MB
- OS - Ubuntu 24.04


## Docker Images Used

| Service | Image | Purpose |
|---------|-------|---------|
| Jenkins | `jenkins:lts-jdk21` | CI/CD Pipeline |
| SonarQube | `sonarqube:community` | Code Quality Analysis |
| Prometheus | `prom/prometheus:latest` | Metrics Collection |
| Grafana | `grafana/grafana:latest` | Monitoring Dashboard |
| OWASP ZAP | `zaproxy/zap-stable` | Security Testing |
| Node Exporter | `prom/node-exporter:latest` | System Metrics |


## File Structure

All files related to infrastructure setup, deployment, monitoring, configurations as needed for the assignment are available in the "infra" folder.

```text
.
├── Jenkinsfile
├── README.md
├── Vagrantfile
├── ansible
│   ├── deploy-to-vm.yml
│   └── inventory
├── auto_script.sh
├── cleanup_script.sh
├── docker-compose.yml
├── grafana
│   └── provisioning
│       ├── dashboards
│       │   ├── basic-dashboard.json
│       │   ├── dashboards.yml
│       │   └── jenkins-dashboard.json
│       └── datasources
│           └── datasources.yml
├── jenkins
│   ├── Dockerfile
│   ├── init.groovy.d
│   │   ├── config-vagrant-ssh.groovy
│   │   └── configure-sonar.groovy
│   ├── jenkins.yaml
│   └── plugins.txt
├── prod_vm_provision.sh
├── prometheus
│   └── prometheus.yml
└── zap
    └── Dockerfile

10 directories, 20 files


```
## Setup

### 1. Clone Repository
```bash
git clone https://github.com/Drakey-504/petclinic-devsecops-pipeline.git
cd petclinic-devsecops-pipeline/infra
```

### 2. Setup Infrastructure and Configure Tools
```bash
// Make bash script executable on host and run
chmod +x ./auto_script.sh
./auto_script.sh
```
The automation script handles setup for all Docker containers, staging and production VMs, and tooling configurations for all services.
Jenkins is also configured via JCasC to have a pipeline already setup to run on logging in. 
The script also sets up credentials required to access Jenkins (based on user input), and credentials required by the Jenkins container to access other services (like Sonarqube, ZAP, etc).
More specifications are available as comments in the auto_script.sh file

## Access URLs & Credentials

### Jenkins
- **URL**: http://localhost:8080
- **Username**: `jenkins_admin`
- **Password**: `devops@2025` (Default. Check auto_script.sh to user define)

### SonarQube
- **URL**: http://localhost:9000
- **Username**: `admin`
- **Password**: `admin`

### Grafana
- **URL**: http://localhost:3000
- **Username**: `admin`
- **Password**: `admin`

### Prometheus
- **URL**: http://localhost:9090
- **No authentication required**

### OWASP ZAP
- **URL**: http://localhost:8090


## Pipeline Steps

### 1. Trigger Build
- Navigate to Jenkins -> Skip setup (If shown) →  Navigate to **spring-petclinic-pipeline** job
- Click **Build Now**

### 2. Pipeline Stages
1. **Checkout** - Git clone
2. **Build** - Build and run unit tests with JUnit
3. **SonarQube Analysis** - Code quality scan
4. **Package** - Maven package
5. **Staging** - Push JAR to Staging VM via Ansible
6. **Security Scan** - OWASP ZAP testing
7. **Deploy to Production** - Final deployment (VM)

### 3. Monitor Progress
- **Build logs**: Jenkins console output
- **Blue Ocean plugin**: To visualize the build process
- **Code quality**: SonarQube dashboard
- **System metrics**: Grafana dashboards
- **Security results**: OWASP ZAP reports

## VM URLs

### Staging Environment
- **URL**: http://192.168.56.201:8081
- **VM**: petclinic-stage

### Production Environment  
- **URL**: http://192.168.56.200:8081
- **VM**: petclinic-prod

Note: The IPs above are hardcoded to be used by Vagrant when setting up the VMs. Therefore, these IPs must be free and available on the host machine's private network.

## Success Verification

Paste the Prod URL above in the browser after successful build. This leads to the homepage for the spring-petclinic application. 

## Cleanup


```bash
// Make bash script executable on host and run
cd petclinic-devsecops-pipeline/infra
chmod +x ./cleanup_script.sh
./cleanup_script.sh
```

This script removes the created docker containers and volumes in the docker compose file. 
Additionally, it removes the Vagrant Prod and Staging VMs and removes local Vagrant state.
Note: The script excepts confirmation from the user before removing Vagrant VMs and will wait indefinitely for user input.
