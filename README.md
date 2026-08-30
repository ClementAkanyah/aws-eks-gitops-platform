\# Tech Challenge 2 — Application Deployment with Docker, Terraform, AWS EKS, Jenkins \& GitOps



A production-style deployment of a containerized Flask application to Amazon EKS using Terraform, Helm, Kubernetes autoscaling, Jenkins CI/CD, GitHub Actions, and Argo CD.



The application returns:



```text

Hello, World!

```



This project implements two deployment approaches:



\- \*\*Main branch:\*\* Jenkins CI/CD

\- \*\*GitOps branch:\*\* GitHub Actions + Argo CD



\---



\## Architecture



!\[Tech Challenge 2 Architecture](docs/images/architecture-diagram.png)



The solution consists of a private GitHub repository, Amazon ECR, Amazon EKS, an EC2-hosted Jenkins server, Helm, Kubernetes, an AWS Application Load Balancer, Horizontal Pod Autoscaling, Cluster Autoscaler, GitHub Actions, and Argo CD.



\### Jenkins CI/CD path



```text

GitHub main branch

&#x20;       ↓

Jenkins

&#x20;       ↓

Docker Build

&#x20;       ↓

Amazon ECR

&#x20;       ↓

Helm / kubectl

&#x20;       ↓

Amazon EKS

&#x20;       ↓

Kubernetes Service / Ingress

&#x20;       ↓

AWS Application Load Balancer

&#x20;       ↓

Internet

```



\### GitOps path



```text

GitHub gitops branch

&#x20;       ↓

GitHub Actions

&#x20;       ↓

Docker Build

&#x20;       ↓

Amazon ECR

&#x20;       ↓

Update Helm image tag

&#x20;       ↓

Git commit

&#x20;       ↓

Argo CD

&#x20;       ↓

Amazon EKS

```



\---



\## Project Objectives



This project satisfies the challenge requirements by implementing:



\- A simple Flask web application displaying `Hello, World!`

\- Docker containerization

\- Amazon ECR for container image storage

\- Terraform Infrastructure as Code

\- Amazon EKS Kubernetes cluster

\- Managed EKS node group using `t3.small`

\- Minimum 1 worker node

\- Maximum 4 worker nodes

\- Kubernetes Metrics Server

\- Horizontal Pod Autoscaler

\- 50% CPU utilization target

\- 50% memory utilization target

\- Minimum 1 application pod

\- Maximum 3 application pods

\- Kubernetes Cluster Autoscaler

\- AWS Load Balancer Controller

\- Internet-facing Application Load Balancer

\- Helm-based Kubernetes deployments

\- Jenkins CI/CD

\- GitHub Actions CI

\- AWS OIDC federation

\- Argo CD continuous delivery

\- Private GitHub repository



\---



\## Technology Stack



| Component | Technology |

|---|---|

| Application | Python Flask |

| Web server | Gunicorn |

| Containerization | Docker |

| Container Registry | Amazon ECR |

| Infrastructure as Code | Terraform |

| Kubernetes | Amazon EKS |

| Kubernetes package manager | Helm |

| Load balancing | AWS Application Load Balancer |

| Pod autoscaling | Kubernetes HPA |

| Node autoscaling | Kubernetes Cluster Autoscaler |

| Metrics | Kubernetes Metrics Server |

| CI/CD | Jenkins |

| GitOps CI | GitHub Actions |

| GitOps CD | Argo CD |

| Authentication | AWS IAM / OIDC / IRSA |

| Source control | GitHub |



\---



\## Repository Structure



```text

tech-challenge-2/

│

├── app/

│   ├── app.py

│   ├── requirements.txt

│   ├── Dockerfile

│   └── .dockerignore

│

├── terraform/

│   ├── providers.tf

│   ├── variables.tf

│   ├── vpc.tf

│   ├── ecr.tf

│   ├── eks.tf

│   ├── iam.tf

│   ├── outputs.tf

│   └── aws-load-balancer-controller-iam-policy.json

│

├── helm/

│   └── hello-world/

│       ├── Chart.yaml

│       ├── values.yaml

│       └── templates/

│           ├── deployment.yaml

│           ├── service.yaml

│           ├── ingress.yaml

│           └── hpa.yaml

│

├── argocd/

│   └── application.yaml

│

├── .github/

│   └── workflows/

│       └── gitops.yml

│

├── docs/

│   └── images/

│

├── Jenkinsfile

├── .gitignore

└── README.md

```



\---



\# Application



The Flask application is intentionally simple because the focus of the challenge is cloud infrastructure, Kubernetes, and CI/CD.



`app/app.py`:



```python

from flask import Flask



app = Flask(\_\_name\_\_)



@app.route("/")

def hello():

&#x20;   return "Hello, World!"



if \_\_name\_\_ == "\_\_main\_\_":

&#x20;   app.run(host="0.0.0.0", port=5000)

```



\---



\## Running Locally



\### Requirements



Install:



\- Git

\- Python 3

\- Docker

\- Terraform

\- AWS CLI

\- kubectl

\- Helm



Clone the repository:



```bash

git clone https://github.com/ClementAkanyah/tech-challenge-2.git

cd tech-challenge-2

```



Create a Python environment:



```bash

python -m venv .venv

```



Install dependencies:



```bash

pip install -r app/requirements.txt

```



Run:



```bash

python app/app.py

```



Open:



```text

http://localhost:5000

```



Expected response:



```text

Hello, World!

```



\---



\# Docker



The application is packaged with a lightweight Python image and served by Gunicorn.



Build:



```bash

docker build -t tech-challenge-2-app:1.0 ./app

```



Run:



```bash

docker run -p 5000:5000 tech-challenge-2-app:1.0

```



Verify:



```text

http://localhost:5000

```



\---



\# Terraform Infrastructure



Terraform provisions the core AWS environment in:



```text

us-east-2

```



The Terraform configuration creates:



\- VPC

\- Internet Gateway

\- Public subnets across multiple Availability Zones

\- Route tables

\- Amazon ECR repository

\- Amazon EKS cluster

\- EKS managed node group

\- IAM roles

\- EKS OIDC provider

\- IRSA roles

\- AWS Load Balancer Controller IAM permissions

\- Cluster Autoscaler IAM permissions

\- Security-related infrastructure



\## Initialize



```bash

cd terraform

terraform init

```



\## Validate



```bash

terraform fmt

terraform validate

```



\## Plan



```bash

terraform plan

```



Terraform initially planned 17 AWS resources.



!\[Terraform Plan](docs/images/terraform-plan.png)



\## Apply



```bash

terraform apply

```



The initial infrastructure deployment completed successfully with:



```text

Resources: 17 added, 0 changed, 0 destroyed

```



!\[Terraform Apply](docs/images/terraform-apply.png)



\---



\# Amazon EKS



The EKS cluster is named:



```text

tech-challenge-2-eks

```



Configure local Kubernetes access:



```bash

aws eks update-kubeconfig \\

&#x20; --region us-east-2 \\

&#x20; --name tech-challenge-2-eks

```



Verify nodes:



```bash

kubectl get nodes -o wide

```



!\[EKS Worker Node](docs/images/eks-node-ready.png)



\---



\# EKS Node Autoscaling



The managed node group uses:



```text

Instance type: t3.small

Minimum nodes: 1

Desired nodes: 1

Maximum nodes: 4

```



Cluster Autoscaler monitors unschedulable workloads and adjusts the node group within those limits.



!\[Cluster Autoscaler](docs/images/cluster-autoscaler.png)



\---



\# Kubernetes Metrics Server



Metrics Server provides CPU and memory utilization data to Kubernetes.



Verification:



```bash

kubectl top nodes

```



This enables HPA to make scaling decisions based on resource utilization.



\---



\# Horizontal Pod Autoscaler



The application HPA is configured with:



```text

Minimum replicas: 1

Maximum replicas: 3

CPU target: 50%

Memory target: 50%

```



The challenge wording refers to pod scaling per node. Kubernetes HPA scales Deployment replicas globally rather than setting a pod count independently on each node. Kubernetes scheduling and topology spread constraints are therefore used to distribute replicas across available nodes.



!\[Kubernetes Workload and HPA](docs/images/kubernetes-workload-hpa.png)



\---



\# AWS Load Balancer Controller



AWS Load Balancer Controller was installed using Helm and configured using IAM Roles for Service Accounts.



It dynamically provisions an AWS Application Load Balancer from the Kubernetes Ingress resource.



!\[AWS Load Balancer Controller](docs/images/aws-load-balancer-controller.png)



\---



\# Kubernetes Deployment with Helm



The application is deployed using a Helm chart containing:



\- Deployment

\- ClusterIP Service

\- ALB Ingress

\- HorizontalPodAutoscaler

\- resource requests and limits

\- readiness probe

\- liveness probe

\- topology spread constraints



Install:



```bash

helm upgrade --install hello-world ./helm/hello-world \\

&#x20; --namespace hello-world \\

&#x20; --create-namespace

```



Verify:



```bash

kubectl get pods -n hello-world

kubectl get service -n hello-world

kubectl get ingress -n hello-world

kubectl get hpa -n hello-world

```



\---



\# Amazon ECR



The Docker image is pushed to:



```text

tech-challenge-2-app

```



Example:



```bash

docker push <AWS\_ACCOUNT\_ID>.dkr.ecr.us-east-2.amazonaws.com/tech-challenge-2-app:1.0

```



!\[Amazon ECR Image Push](docs/images/ecr-image-push.png)



\---



\# Application Load Balancer



The Kubernetes Ingress provisions an internet-facing ALB.



```bash

kubectl get ingress -n hello-world

```



!\[ALB Ingress](docs/images/alb-ingress.png)



A direct HTTP verification returned:



```text

StatusCode: 200

Content: Hello, World!

```



!\[Application HTTP Verification](docs/images/application-http-verification.png)



\---



\# Public Application



The application is publicly accessible through the ALB:



```text

http://k8s-hellowor-hellowor-de6f09c274-1459841893.us-east-2.elb.amazonaws.com

```



Expected output:



```text

Hello, World!

```



!\[Public Hello World Application](docs/images/application-browser.png)



\---



\# Jenkins CI/CD



The primary CI/CD implementation lives on the `main` branch.



Jenkins runs on an Amazon EC2 `t3.medium` instance.



!\[Jenkins EC2](docs/images/jenkins-ec2-instance.png)



The Jenkins host includes:



\- Java

\- Jenkins

\- Git

\- Docker

\- AWS CLI

\- kubectl

\- Helm



AWS authentication is handled through an EC2 IAM instance profile rather than permanent access keys.



The Jenkins server also uses a dedicated read-only GitHub deploy key for repository access.



!\[Jenkins Dashboard](docs/images/jenkins-dashboard.png)



\---



\## Jenkins Pipeline



The `Jenkinsfile` performs:



```text

Checkout

&#x20;  ↓

Docker Build

&#x20;  ↓

Amazon ECR Login

&#x20;  ↓

Tag Image

&#x20;  ↓

Push Image

&#x20;  ↓

Configure kubectl

&#x20;  ↓

Helm Upgrade / Install

&#x20;  ↓

Wait for Kubernetes Rollout

&#x20;  ↓

Verify Pods / Service / Ingress / HPA

```



The pipeline completed successfully.



!\[Jenkins Pipeline Success](docs/images/jenkins-pipeline-success.png)



The final verification stage confirmed:



\- application pod running

\- Kubernetes Service available

\- ALB Ingress available

\- HPA configured

\- pipeline completed successfully



!\[Jenkins Deployment Verification](docs/images/jenkins-deployment-verification.png)



\---



\# GitOps Alternative



The GitOps implementation is maintained separately on:



```text

gitops

```



The workflow uses:



```text

GitHub Actions → Amazon ECR → Helm values update → Git commit → Argo CD → Amazon EKS

```



This keeps the original Jenkins implementation on `main` separate from the modern GitOps implementation.



\---



\# GitHub Actions



GitHub Actions performs:



1\. Checkout

2\. AWS authentication using OIDC

3\. ECR authentication

4\. Docker image build

5\. Docker image push

6\. Update Helm image tag

7\. Commit the new desired state to `gitops`



No long-lived AWS access keys are stored in GitHub.



GitHub authenticates with AWS by assuming an IAM role through:



```text

token.actions.githubusercontent.com

```



The workflow completed successfully:



!\[GitHub Actions Success](docs/images/github-actions-success.png)



\---



\# Argo CD



Argo CD runs inside the EKS cluster.



It monitors:



```text

gitops branch

helm/hello-world

```



Automated synchronization is configured with:



```yaml

syncPolicy:

&#x20; automated:

&#x20;   prune: true

&#x20;   selfHeal: true

```



Argo CD reached:



```text

SYNC STATUS: Synced

HEALTH STATUS: Healthy

```



!\[Argo CD Synced Healthy](docs/images/argocd-synced-healthy.png)



The running Kubernetes Deployment was automatically updated to the commit-tagged image generated by GitHub Actions.



!\[GitOps Image Deployment](docs/images/gitops-image-deployment.png)



\---



\# Security Design



Security decisions implemented in this challenge include:



\- Private GitHub repository

\- Dedicated read-only Jenkins deploy key

\- Dedicated read-only Argo CD deploy key

\- EC2 IAM instance role for Jenkins

\- EKS OIDC provider

\- IAM Roles for Service Accounts

\- GitHub Actions OIDC federation

\- No permanent AWS access keys in GitHub Actions

\- No AWS credentials stored in Jenkinsfile

\- Terraform state excluded from Git

\- `.pem`, `.key`, `.env`, and temporary IAM policy files excluded from Git

\- ECR image scanning enabled

\- Security group-controlled network access



\---



\# Troubleshooting



Several real deployment issues were identified and resolved during implementation.



\## Docker Desktop Engine Unavailable



Initial Docker builds failed because the Docker CLI could not connect to:



```text

dockerDesktopLinuxEngine

```



Resolution:



Docker Desktop was started before rerunning the image build.



\---



\## PowerShell ECR Authentication



Piping the ECR token directly into Docker returned:



```text

400 Bad Request

```



The token was instead retrieved first and then used for authentication.



\---



\## Jenkins → EKS API Timeout



Jenkins initially received:



```text

dial tcp 10.20.x.x:443: i/o timeout

```



The Jenkins security group was granted access to the EKS cluster security group on TCP 443.



\---



\## Jenkins Kubernetes Forbidden



After network connectivity was restored, Jenkins reached EKS but received:



```text

nodes is forbidden

```



The EKS authentication mode was updated to:



```text

API\_AND\_CONFIG\_MAP

```



An EKS access entry and access policy were then associated with the Jenkins IAM role.



\---



\## Jenkins GitHub Host Verification



Jenkins initially failed to access GitHub with:



```text

Host key verification failed

```



GitHub's SSH host key was added to Jenkins' `known\_hosts`.



A dedicated repository deploy key was then created to resolve:



```text

Permission denied (publickey)

```



\---



\## Public ALB Browser Timeout



The application initially appeared unreachable in Chrome.



Validation using:



```powershell

Invoke-WebRequest

Test-NetConnection

```



confirmed:



```text

HTTP 200 OK

TcpTestSucceeded: True

```



The ALB and Kubernetes application were functioning correctly; the issue was isolated to browser behavior.



\---



\## GitHub Actions OIDC Failure



The first GitHub Actions run failed with:



```text

Not authorized to perform sts:AssumeRoleWithWebIdentity

```



The IAM role trust policy was updated to correctly match the GitHub repository OIDC subject.



The rerun completed successfully.



\---



\# Branch Strategy



\## `main`



Contains the primary challenge implementation:



```text

Jenkins

Docker

Terraform

EKS

Helm

Kubernetes

ALB

HPA

Cluster Autoscaler

```



\## `gitops`



Contains the GitOps alternative:



```text

GitHub Actions

AWS OIDC

Amazon ECR

Argo CD

Helm

EKS

```



\---



\# Reproducing the Environment



\## 1. Clone



```bash

git clone https://github.com/ClementAkanyah/tech-challenge-2.git

cd tech-challenge-2

```



\## 2. Authenticate AWS CLI



```bash

aws sts get-caller-identity

```



\## 3. Provision infrastructure



```bash

cd terraform

terraform init

terraform validate

terraform plan

terraform apply

```



\## 4. Configure kubectl



```bash

aws eks update-kubeconfig \\

&#x20; --region us-east-2 \\

&#x20; --name tech-challenge-2-eks

```



\## 5. Build and push application image



```bash

docker build -t tech-challenge-2-app:1.0 ./app

```



Authenticate to Amazon ECR and push the image.



\## 6. Install platform components



Install:



\- Kubernetes Metrics Server

\- AWS Load Balancer Controller

\- Cluster Autoscaler



\## 7. Deploy application



```bash

helm upgrade --install hello-world ./helm/hello-world \\

&#x20; --namespace hello-world \\

&#x20; --create-namespace

```



\## 8. Verify



```bash

kubectl get nodes

kubectl get pods -n hello-world

kubectl get hpa -n hello-world

kubectl get ingress -n hello-world

```



\---



\# Verification Summary



| Requirement | Result |

|---|---|

| Flask Hello World application | ✅ |

| Docker image | ✅ |

| Amazon ECR | ✅ |

| Terraform | ✅ |

| Amazon EKS | ✅ |

| `t3.small` worker nodes | ✅ |

| Node scaling 1 → 4 | ✅ |

| Kubernetes HPA | ✅ |

| CPU target 50% | ✅ |

| Memory target 50% | ✅ |

| Pod scaling 1 → 3 | ✅ |

| Metrics Server | ✅ |

| AWS ALB | ✅ |

| Helm deployment | ✅ |

| Jenkins CI/CD | ✅ |

| GitHub Actions | ✅ |

| AWS OIDC | ✅ |

| Argo CD | ✅ |

| GitOps automated sync | ✅ |

| Public application | ✅ |



\---



\# Cleanup



AWS EKS and associated infrastructure generate ongoing charges.



After grading, Terraform-managed resources can be removed with:



```bash

cd terraform

terraform destroy

```



Resources created outside Terraform, such as the Jenkins EC2 instance and supporting manually-created IAM resources, should also be removed after grading.



\---



\# Final Result



The project demonstrates a complete cloud-native application delivery platform with:



```text

Docker

Terraform

AWS EKS

Kubernetes

Helm

HPA

Cluster Autoscaler

AWS ALB

Amazon ECR

Jenkins

GitHub Actions

Argo CD

AWS IAM/OIDC

```



Both CI/CD approaches successfully deploy the same application to EKS, and the application is publicly accessible through an AWS Application Load Balancer.



```text

Hello, World!

```

