# Testing Guide for Three-Tier CI/CD Pipelines

This guide provides commands and expected outputs for manual testing of the adapted pipelines on the Jenkins server (192.168.1.100).

## Prerequisites for Testing
- Jenkins running on 192.168.1.100.
- kubectl configured on Jenkins with access to K8s cluster.
- Local registry accessible at 192.168.1.100:5000.
- Repository cloned or accessible.

## Setting Up Jenkins Jobs
1. Log into Jenkins web UI (http://192.168.1.100:8080).
2. Create new Pipeline jobs for each:
   - Job 1: "Three-Tier-Backend" - Use Jenkinsfile-Backend.
   - Job 2: "Three-Tier-Frontend" - Use Jenkinsfile-Frontend.
   - Job 3: "Three-Tier-Database" - Use Jenkinsfile-Database.
3. For each job, set Pipeline script from SCM, Git, URL: https://github.com/kumbharnv/End-to-End-Kubernetes-Three-Tier-DevSecOps-Project.git, Script Path: Jenkins-Pipeline-Code/Jenkinsfile-Backend (etc.).

## Manual Testing Commands and Expected Outputs

### 1. Test Database Pipeline (Jenkinsfile-Database)
- **Trigger**: Run the "Three-Tier-Database" job in Jenkins web UI.
- **Expected Stages and Outputs**:
  - Cleaning Workspace: Workspace cleaned.
  - Checkout from Git: Repository cloned successfully.
  - Deploy Database to Kubernetes: `kubectl apply -f Kubernetes-Manifests-file/Database/ -n three-tier`
    - Expected Output: `deployment.apps/mongodb created`, `service/mongodb-svc created`, `persistentvolume/mongo-pv created`, `persistentvolumeclaim/mongo-volume-claim created`, `secret/mongo-sec created`.
- **Verification Command** (run on Jenkins or K8s master):
  ```
  kubectl get pods -n three-tier
  ```
  - Expected Output: `mongodb-xxxxxxxxx-xxxxx   1/1     Running   0          xxm`

### 2. Test Backend Pipeline (Jenkinsfile-Backend)
- **Trigger**: Run the "Three-Tier-Backend" job in Jenkins web UI.
- **Expected Stages and Outputs**:
  - Cleaning Workspace: Workspace cleaned.
  - Checkout from Git: Repository cloned.
  - Sonarqube Analysis: Analysis completed, quality gate passed (or failed if configured).
  - Quality Check: OK or failed based on gate.
  - OWASP Dependency-Check Scan: Scan completed, report generated.
  - Trivy File Scan: Vulnerabilities scanned, output to trivyfs.txt.
  - Docker Image Build: `docker build -t backend .` - Build successful.
  - Local Registry Image Pushing: `docker tag backend 192.168.1.100:5000/backend:${BUILD_NUMBER}`, `docker push 192.168.1.100:5000/backend:${BUILD_NUMBER}` - Push successful.
  - TRIVY Image Scan: Image scanned, output to trivyimage.txt.
  - Deploy to Kubernetes: `kubectl set image deployment/api api=192.168.1.100:5000/backend:${BUILD_NUMBER} -n three-tier`
    - Expected Output: `deployment.apps/api image updated`
- **Verification Commands**:
  ```
  kubectl get pods -n three-tier | grep api
  kubectl logs -l role=api -n three-tier --tail=10
  ```
  - Expected Output: Pod running, logs show backend starting (e.g., "Server running on port 3500").

### 3. Test Frontend Pipeline (Jenkinsfile-Frontend)
- **Trigger**: Run the "Three-Tier-Frontend" job in Jenkins web UI.
- **Expected Stages and Outputs**: Similar to Backend, but for frontend.
  - Docker Image Build: `docker build -t frontend .`
  - Local Registry Image Pushing: Push to 192.168.1.100:5000/frontend:${BUILD_NUMBER}
  - Deploy to Kubernetes: `kubectl set image deployment/frontend frontend=192.168.1.100:5000/frontend:${BUILD_NUMBER} -n three-tier`
- **Verification Commands**:
  ```
  kubectl get pods -n three-tier | grep frontend
  kubectl logs -l role=frontend -n three-tier --tail=10
  ```
  - Expected Output: Pod running, logs show React app starting.

### 4. End-to-End Verification
- **Check All Pods**:
  ```
  kubectl get pods -n three-tier
  ```
  - Expected: All pods (mongodb, api, frontend) in Running state.
- **Check Services**:
  ```
  kubectl get services -n three-tier
  ```
  - Expected: mongodb-svc, api-svc, frontend-svc listed.
- **Test API** (from Jenkins or another node):
  ```
  curl http://<backend-service-ip>:3500/healthz
  ```
  - Expected: JSON response like {"status": "ok"} or similar.
- **Test Frontend** (access via ingress or service IP on port 3000).
  - Expected: React app loads, connects to backend.

## Troubleshooting
- If registry push fails: Ensure 192.168.1.100:5000 is accessible; check Docker daemon config for insecure registries.
- If K8s deployment fails: Verify kubectl config; check namespace and secrets.
- If scans fail: Ensure tools are installed and configured in Jenkins.

Run the jobs in order: Database first, then Backend, then Frontend. Monitor Jenkins console output for each stage.
