# 🚀🛠️ Designing a 10-Microservices Application Deployment on EKS! 🤖

![](https://miro.medium.com/v2/resize:fit:736/1*BR95t3O5WYZykgPvJnv2HA.png)

In the realm of cloud computing, deploying a complex application with precision and efficiency is a paramount challenge. Amazon Web Services (AWS) offers Elastic Kubernetes Service (EKS), a fully managed Kubernetes service that simplifies the deployment, management, and scaling of containerized applications. In this project, we'll delve into the intricacies of orchestrating a 10-tier application on AWS EKS.

## Files in this folder

| File | Purpose |
|------|---------|
| `Jenkinsfile` | Full pipeline: checkout → SonarQube → build & push 11 images → deploy to EKS |
| `sa.yaml` | ServiceAccount `jenkins` in namespace `webapps` |
| `rol.yaml` | Role granting the pipeline its Kubernetes permissions |
| `bind.yaml` | RoleBinding attaching the Role to the ServiceAccount |
| `sec.yaml` | Long-lived ServiceAccount token Secret (see the 1.24+ note) |

> ℹ️ The application manifests (`deployment-service.yml` with all 10 microservices) live in the source repository [SushantOps/10-Tier-MicroService-Appliction](https://github.com/SushantOps/10-Tier-MicroService-Appliction) — they are checked out by the pipeline, not stored here.

## **Designing the 10-Microservices Application**

## **1\. Presentation Tier 🖥️:**

* Host your frontend application here.

* Utilize AWS Elastic Load Balancer (ELB) for distributing incoming traffic.

## **2\. Web Server Tier 🌐:**

* Deploy web servers using containers managed by Kubernetes pods.

* Leverage Kubernetes Services for load balancing within the web server tier.

## **3\. Application Tier 🚀:**

* Run application logic in containers using Kubernetes Deployments.

* Implement auto-scaling based on demand to ensure optimal performance. See [Autoscaling (HPA)](../../learning/Autoscaling-HPA/README.md).

## **4\. API Gateway Tier 🚪:**

* Use Amazon API Gateway to create, publish, and manage APIs.

* Enable authentication and throttling for secure and controlled access.

## **5\. Business Logic Tier 💼:**

* Containerize business logic components and deploy using Kubernetes.

* Utilize AWS Lambda for serverless execution of specific functions.

## **6\. Message Queue Tier 📬:**

* Implement message queues like Amazon Simple Queue Service (SQS) for asynchronous communication between components.

* Ensure reliability and scalability of message processing.

## **7\. Data Access Tier 📊:**

* Manage data storage and retrieval using Amazon Relational Database Service (RDS) or Amazon DynamoDB.

* Implement read replicas for scalability and fault tolerance.

## **8\. Caching Tier 🚀:**

* Utilize Amazon ElastiCache for in-memory caching to enhance application performance.

* Configure caching strategies based on the application's needs.

## **9\. Storage Tier 🗄️:**

* Use Amazon S3 for scalable and durable object storage.

* Implement data lifecycle policies to manage data efficiently.

## **10\. Infrastructure Tier 🌐:**

* Leverage Amazon EKS to manage and orchestrate containers effectively.

* Utilize AWS Identity and Access Management (IAM) for secure access control.

![](https://miro.medium.com/v2/resize:fit:736/0*lBtxr2c2uMcXmz9y.png)

# **Step-by-Step Implementation :-**

1. **Create an AWS EC2 instance**

![](https://miro.medium.com/v2/resize:fit:736/0*jetGaqyeE1uAXzuC.png)

**2\. Create a user and give the following permissions :-**

![](https://miro.medium.com/v2/resize:fit:736/1*giZzF4Cn9X49DlLdCexjbg.png)

Connect with the EC2 instance you can use ssh or Putty or MobaXterm.

**3\. After connecting, install the AWS CLI on your server and configure your credentials**
[https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

**4\. After connecting install Jenkins on your server**
[https://www.jenkins.io/doc/book/installing/linux/#debianubuntu](https://www.jenkins.io/doc/book/installing/linux/#debianubuntu)

**5\. Now install kubectl on the Linux server**
[https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-kubectl-binary-with-curl-on-linux](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-kubectl-binary-with-curl-on-linux)

**6\. Install eksctl**
[https://docs.aws.amazon.com/emr/latest/EMR-on-EKS-DevelopmentGuide/setting-up-eksctl.html](https://docs.aws.amazon.com/emr/latest/EMR-on-EKS-DevelopmentGuide/setting-up-eksctl.html)

**7\. Install docker and give permission**

```bash
sudo apt-get install docker.io
sudo usermod -aG docker ubuntu
sudo newgrp docker
```

**8\. Install SonarQube from docker image**

```bash
docker run -d -p 9000:9000 sonarqube:lts-community
```

**9\. Install EKS**

```bash
eksctl create cluster --name=my-eks2 \
        --region=ap-south-1 \
        --zones=ap-south-1a,ap-south-1b \
        --without-nodegroup

# after the above setup completes, run this
eksctl utils associate-iam-oidc-provider \
 --region ap-south-1 \
 --cluster my-eks2 \
 --approve

eksctl create nodegroup --cluster=my-eks2 \
   --region=ap-south-1 \
   --name=node2 \
   --node-type=t3.medium \
   --nodes=3 \
   --nodes-min=2 \
   --nodes-max=3 \
   --node-volume-size=20 \
   --ssh-public-key=10-tier-key \
   --managed \
   --asg-access \
   --external-dns-access \
   --full-ecr-access \
   --appmesh-access \
   --alb-ingress-access
```

**10\. Install these plugins in Jenkins** (Manage Jenkins → Plugins → Available Plugins):

```text
SonarQube Scanner
Docker
Docker Pipeline
Docker Commons
CloudBees Docker Build and Publish
Kubernetes
Kubernetes CLI
```

**Configure Sonar Server in Manage Jenkins**

Grab the Public IP Address of your EC2 Instance — SonarQube works on Port 9000, so open `<Public IP>:9000`. Go to your SonarQube Server. Click on Administration → Security → Users → Click on Tokens and Update Token → Give it a name → and click on Generate Token

![](https://miro.medium.com/v2/resize:fit:736/0*ohY_aQ9IHj9FQ1hl.png)

Click on Update Token

![](https://miro.medium.com/v2/resize:fit:736/0*0DBl74AeobYkTgyy.png)

**Copy this Token**

Go to Dashboard → Manage Jenkins → Credentials → Add Secret Text. It should look like this

![](https://miro.medium.com/v2/resize:fit:736/0*uVqbfa70yVgMLcFt.png)

Now, go to Dashboard → Manage Jenkins → Configure System

![](https://miro.medium.com/v2/resize:fit:736/0*TGpOriGYMoFi8w0j.png)

Click on Apply and Save

**Configure System option** is used in Jenkins to configure different servers

**Global Tool Configuration** is used to configure different tools that we install using Plugins

We will install sonar-scanner in tools.

![](https://miro.medium.com/v2/resize:fit:736/0*mpd0x6wSOIxuHZ58.png)

**11\. Go to the EKS service and add all traffic to its SG**

![](https://miro.medium.com/v2/resize:fit:736/1*fCoAzi74QtB4S1c7q96OlQ.png)

> ⚠️ "All traffic" on the node security group is a lab shortcut only. In production, restrict inbound rules to what your services actually need — see [Network Policies](../../learning/Network-Policies-in-K8s/README.md) for in-cluster traffic control on top of that.

**12\. Create a Service Account, Role and RoleBinding in the `webapps` namespace, then a token Secret**

All four manifests are in this folder — apply them in order:

```bash
kubectl create namespace webapps
kubectl apply -f sa.yaml      # ServiceAccount
kubectl apply -f rol.yaml     # Role
kubectl apply -f bind.yaml    # RoleBinding
kubectl apply -f sec.yaml     # token Secret (see note below)
```

Creating Service Account

1\. create namespace:-

![](https://miro.medium.com/v2/resize:fit:663/1*PqFe-i3Yia1gqDD-1bZ-dg.png)

2\. `sa.yaml` — ServiceAccount `jenkins` in namespace `webapps` (file is in this folder):

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: webapps
  labels:
    app: jenkins
    environment: dev
```

3\. Now we create the Role (`rol.yaml`). The pipeline needs full CRUD on the workload resources **in the `webapps` namespace**:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-role
  namespace: webapps
rules:
  - apiGroups: [""]
    resources: [pods, pods/log, configmaps, events, endpoints,
                persistentvolumeclaims, serviceaccounts, services]
    verbs: [get, list, watch, create, update, patch, delete]
  - apiGroups: ["apps"]
    resources: [deployments, daemonsets, replicasets, statefulsets]
    verbs: [get, list, watch, create, update, patch, delete]
  - apiGroups: ["autoscaling"]
    resources: [horizontalpodautoscalers]
    verbs: [get, list, watch, create, update, patch, delete]
  - apiGroups: ["batch"]
    resources: [jobs, cronjobs]
    verbs: [get, list, watch, create, update, patch, delete]
  - apiGroups: ["networking.k8s.io"]
    resources: [ingresses]          # note: "ingresses", not "ingress"
    verbs: [get, list, watch, create, update, patch, delete]
  - apiGroups: ["policy"]
    resources: [poddisruptionbudgets]
    verbs: [get, list, watch, create, update, patch, delete]
```

> ⚠️ RBAC gotchas this Role fixes vs. older versions of this guide:
> * resources must be listed **per apiGroup** — `deployments` does not exist in the core group
> * `nodes`, `namespaces` and `persistentvolumes` are cluster-scoped — a namespaced Role cannot grant them, so listing them here does nothing

4\. Now assign the role to the service account (`bind.yaml`):

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-rolebinding
  namespace: webapps
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: app-role
subjects:
  - kind: ServiceAccount
    name: jenkins
    namespace: webapps
```

![](https://miro.medium.com/v2/resize:fit:736/0*vrx020cTWLWNSrcE.png)

5\. Now create a token for the service account (`sec.yaml`):

```yaml
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: mysecretname
  namespace: webapps
  annotations:
    kubernetes.io/service-account.name: jenkins
  labels:
    app: jenkins
```

![](https://miro.medium.com/v2/resize:fit:569/0*8T2cSyNTgW6u2oY-.png)

> 💡 Since Kubernetes 1.24, long-lived token Secrets are no longer created automatically and are discouraged. This Secret still works, but for a time-boxed token you can skip it and run `kubectl -n webapps create token jenkins` instead.

Go to Jenkins and add a pipeline

![](https://miro.medium.com/v2/resize:fit:736/0*EcuT0dEzhK1ccOV0.png)

Lets goto our Pipeline and add the script below — the same script is in this folder as `Jenkinsfile` (with the workspace path and EKS endpoint fixed to be portable):

```groovy
pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'latest', url: 'https://github.com/SushantOps/10-Tier-MicroService-Appliction.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=10-Tier \
                        -Dsonar.projectName=10-Tier \
                        -Dsonar.java.binaries=.'''
                }
            }
        }

        stage('Build & Push Docker Images') {
            parallel {
                stage('adservice') { steps { buildAndPush('adservice') } }
                stage('cartservice') { steps { buildAndPush('cartservice/src') } }
                stage('checkoutservice') { steps { buildAndPush('checkoutservice') } }
                stage('currencyservice') { steps { buildAndPush('currencyservice') } }
                stage('emailservice') { steps { buildAndPush('emailservice') } }
                stage('frontend') { steps { buildAndPush('frontend') } }
                stage('loadgenerator') { steps { buildAndPush('loadgenerator') } }
                stage('paymentservice') { steps { buildAndPush('paymentservice') } }
                stage('productcatalogservice') { steps { buildAndPush('productcatalogservice') } }
                stage('recommendationservice') { steps { buildAndPush('recommendationservice') } }
                stage('shippingservice') { steps { buildAndPush('shippingservice') } }
            }
        }

        stage('K8s Deploy') {
            steps {
                withKubeConfig(
                    caCertificate: '',
                    clusterName: 'my-eks2',
                    contextName: '',
                    credentialsId: 'k8-token',
                    namespace: 'webapps',
                    restrictKubeConfigAccess: false,
                    serverUrl: 'https://REPLACE-WITH-YOUR-EKS-ENDPOINT.gr7.ap-south-1.eks.amazonaws.com'
                ) {
                    sh 'kubectl apply -f deployment-service.yml'
                    sh 'kubectl get pods'
                    sh 'kubectl get svc'
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline execution complete!"
        }
    }
}

def buildAndPush(String servicePath) {
    script {
        def imageName = "sushantkapare1717/${servicePath.tokenize('/')[-1]}:latest"
        withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
            dir("src/${servicePath}") {
                sh "docker build -t ${imageName} ."
                sh "docker push ${imageName}"
                sh "docker rmi ${imageName}"
            }
        }
    }
}
```

How to get the token value for the `k8-token` credential:

```bash
# from the long-lived Secret (sec.yaml):
kubectl -n webapps get secret mysecretname -o jsonpath='{.data.token}' | base64 -d

# or generate a short-lived one directly (K8s >= 1.24):
kubectl -n webapps create token jenkins
```

Now run the pipeline

After that we run that command

```bash
kubectl get pods -n webapps
```

![](https://miro.medium.com/v2/resize:fit:736/0*ZQKaW4Pfn340JyXr.png)

![](https://miro.medium.com/v2/resize:fit:736/1*sYrGLLwBtA9t3gSIQG9VRA.png)

![](https://miro.medium.com/v2/resize:fit:736/1*HM2SUQOIjgQ-YmCCZr7mSQ.png)

## Teardown

- **Terminate the AWS EC2 Instance**

- **Lastly, do not forget to delete the EKS cluster** (it keeps costing money even when idle):

```bash
eksctl delete cluster --name my-eks2 --region ap-south-1
```

## If you like this article, please share with others. ❤️
