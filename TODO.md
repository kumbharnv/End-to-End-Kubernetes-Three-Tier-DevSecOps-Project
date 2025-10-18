# TODO: Adapt CI/CD Pipelines for Local VMware Kubernetes Setup

## Tasks
- [x] Update Kubernetes-Manifests-file/Backend/deployment.yaml: Change image to local registry and remove imagePullSecrets
- [x] Update Kubernetes-Manifests-file/Frontend/deployment.yaml: Change image to local registry and remove imagePullSecrets
- [x] Modify Jenkins-Pipeline-Code/Jenkinsfile-Backend: Adapt for local registry, remove AWS dependencies, add K8s deployment stage
- [x] Modify Jenkins-Pipeline-Code/Jenkinsfile-Frontend: Adapt for local registry, remove AWS dependencies, add K8s deployment stage
- [x] Create Jenkins-Pipeline-Code/Jenkinsfile-Database: New pipeline for deploying database manifests to K8s
