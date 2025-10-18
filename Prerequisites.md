# Prerequisites and Requirements for End-to-End Kubernetes Three-Tier DevSecOps Project

This document outlines the prerequisites and requirements to set up and run the three-tier CI/CD pipelines for the local VMware Kubernetes environment.

## Infrastructure Requirements

### Network Configuration
- **Jenkins Server (192.168.1.100)**: Running Jenkins, SonarQube (community edition), local Docker registry (registry:2), and required tools.
- **Kubernetes Master (192.168.1.101)**: Kubeadm-installed Kubernetes master node with kubectl.
- **Kubernetes Worker-01 (192.168.1.102)**: Kubeadm-installed worker node.
- **Kubernetes Worker-02 (192.168.1.103)**: Kubeadm-installed worker node.
- All nodes must be accessible via SSH from Jenkins for deployment (if needed), but primarily kubectl access.

### Kubernetes Cluster
- Kubernetes version: Compatible with Kubeadm (e.g., v1.24+).
- CNI: Flannel installed and configured.
- Namespace: `three-tier` created in the cluster.
- Secrets: `mongo-sec` secret for MongoDB credentials must be created in `three-tier` namespace.
- Persistent Volume and Claim: For MongoDB data persistence.

## Software Requirements

### On Jenkins Server (192.168.1.100)
- **Jenkins**: Latest LTS version installed and running.
- **Docker**: Installed and running, with access to local registry at 192.168.1.100:5000.
- **SonarQube**: Running as Docker container (sonarqube:community), accessible at localhost or configured in Jenkins.
- **kubectl**: Installed and configured with kubeconfig pointing to the K8s cluster (master at 192.168.1.101).
- **Trivy**: Installed for container scanning.
- **OWASP Dependency-Check**: Installed as Jenkins plugin or tool.
- **JDK and Node.js**: Installed as Jenkins tools.
- **Git**: Installed for cloning repositories.

### Jenkins Plugins and Configurations
- **Credentials**: 
  - `GITHUB`: GitHub credentials for cloning the repository.
  - `sonar-token`: Token for SonarQube access.
- **Tools**:
  - JDK (configured as 'jdk').
  - Node.js (configured as 'nodejs').
  - SonarQube Scanner (configured as 'sonar-scanner').
  - OWASP Dependency-Check (configured as 'DP-Check').
- **SonarQube Server**: Configured as 'sonar-server' in Jenkins.

### Local Docker Registry
- Running on 192.168.1.100:5000 (registry:2).
- Accessible from Jenkins and K8s nodes without authentication (insecure registry if needed).
- If authentication is required, configure credentials in Jenkins.

### Application Code
- Repository: https://github.com/kumbharnv/End-to-End-Kubernetes-Three-Tier-DevSecOps-Project.git
- Backend: Node.js application in `Application-Code/backend/`.
- Frontend: React application in `Application-Code/frontend/`.
- Database: MongoDB (no code, just manifests).

## Testing and Verification Steps

### Thorough Testing Areas
1. **Pipeline Execution**:
   - Run Jenkinsfile-Backend: Verify code checkout, SonarQube analysis, OWASP scan, Trivy scans, Docker build/push to local registry, and K8s deployment update.
   - Run Jenkinsfile-Frontend: Same as backend, but for frontend deployment.
   - Run Jenkinsfile-Database: Verify database manifests apply to K8s.

2. **Kubernetes Deployments**:
   - Check pods, services, and ingress in `three-tier` namespace.
   - Verify backend API accessible (e.g., health endpoints).
   - Verify frontend serves correctly.
   - Verify MongoDB connectivity from backend.

3. **End-to-End Application**:
   - Access frontend via ingress/service.
   - Test full three-tier functionality (frontend -> backend -> database).

### Common Issues and Fixes
- **Registry Access**: Ensure 192.168.1.100:5000 is reachable; add to insecure registries if needed.
- **K8s Access**: Verify kubeconfig on Jenkins; test `kubectl get nodes`.
- **Image Pull**: If imagePullSecrets were removed, ensure registry is accessible without auth.
- **Ports**: Ensure K8s services expose correct ports (backend: 3500, frontend: 3000, MongoDB: 27017).

## Setup Instructions
1. Ensure all infrastructure is running and networked.
2. Install required software on Jenkins.
3. Configure Jenkins credentials and tools.
4. Clone the repository and run pipelines manually or via webhooks.
5. Monitor logs for errors and adjust configurations as needed.

For any issues, refer to Jenkins logs, K8s events (`kubectl get events -n three-tier`), and application logs.
