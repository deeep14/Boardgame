# Boardgame Application on Kubernetes 🎲

This project demonstrates the deployment of a **Java-based board game application** on **AWS EKS** using a **Jenkins CI/CD pipeline**.  

The pipeline automates building, testing, packaging, containerization, pushing to AWS ECR, and deploying to Kubernetes. It also integrates **SonarQube for code quality**, **Nexus for artifact management**, and **Prometheus for monitoring & alerting**.

---

## 🛠️ Tech Stack

- **Application**: Java (Maven build)
- **CI/CD**: Jenkins
- **Code Quality**: SonarQube
- **Artifact Repository**: Nexus
- **Containerization**: Docker
- **Container Registry**: AWS Elastic Container Registry (ECR)
- **Orchestration**: Amazon Elastic Kubernetes Service (EKS)
- **Monitoring & Alerting**: Prometheus + ServiceMonitor + Alertmanager

---

## 📂 Project Structure
├── Jenkinsfile # CI/CD pipeline definition
├── deployment-service.yaml # Deployment + Service manifest
├── boardgame-servicemonitor.yaml # Prometheus ServiceMonitor
├── boardgame-cpu-alert.yaml # Prometheus alert rules
└── src/ # Java source code


---

## ⚙️ Jenkins Pipeline Stages

1. **Build** → Compiles the Java application using Maven.  
2. **Test** → Runs unit tests.  
3. **Package** → Packages the app into a JAR.  
4. **SonarQube Analysis** → Runs static code analysis.  
5. **Upload to Nexus** → Pushes built JAR to Nexus repository.  
6. **Build Docker Image** → Creates a Docker image for the app.  
7. **Push to ECR** → Pushes the Docker image to AWS ECR.  
8. **Deploy to Kubernetes** → Applies K8s manifests (Deployment, Service, Monitoring, Alerting).  

---




## 🚀 Deployment on EKS

Cluster was created using [**eksctl**](https://eksctl.io/).

Command used to create the cluster and node group -

eksctl create cluster --name k8s-workshop-eks-cluster --region us-east-2 --nodegroup-name ng-default --node-type m7i-flex.large --nodes 2 --nodes-min 1 --nodes-max 3 --managed

The deployment manifest runs **2 replicas** of the app and exposes it via a **LoadBalancer Service** on port `8080`.


## 📊 Monitoring Setup

The app exposes Prometheus metrics at /actuator/prometheus.

ServiceMonitor scrapes metrics from the app every 30s.

PrometheusRule defines an alert for High CPU usage (>80% for 2 minutes).


## 🔑 Prerequisites

AWS CLI configured with appropriate permissions

eksctl installed

kubectl installed

Docker installed

Jenkins with required plugins:

Pipeline

SonarQube Scanner

Nexus Artifact Uploader

AWS Credentials Binding

## ▶️ How to Run

Clone the repository.

Set up Jenkins with required credentials:

nexus-creds (Nexus username/password)

aws-creds (AWS Access Key/Secret)

SONAR_AUTH_TOKEN (SonarQube token)

Push changes to GitHub → Jenkins pipeline triggers automatically.

Access the app via the LoadBalancer DNS after deployment.

✨ With this setup, every code change flows through CI/CD → Quality checks → Artifact storage → Docker → EKS → Monitoring & Alerts.