# Deploy-the-2048-game-on-EKS

## Why we use EKS over Kubeadm and Kops 

### Kubeadm

Kubeadm is a toolkit for bootstrapping a minimum viable Kubernetes cluster that conforms to best practices. It is designed to be a simple, fast way to get up and running with Kubernetes.

#### Key Features
- **Ease of Use:** Kubeadm simplifies the process of setting up a Kubernetes cluster by handling the complex parts of the installation and configuration.
- **Control Plane Setup:** It sets up the essential components of the control plane, such as the API server, scheduler, and controller manager.
- **Node Joining:** Provides straightforward commands for adding new nodes to the cluster.
- **Configuration:** Users have the flexibility to configure networking, storage, and other cluster aspects manually, providing full control over the setup.

#### Pros and Cons
- **Pros:** High flexibility, full control over the cluster, ideal for custom setups.
- **Cons:** Requires manual setup and maintenance, more suited for users with Kubernetes expertise.

### Kops

Kops (Kubernetes Operations) is an open-source project that helps you create, upgrade, and maintain production-grade, highly available Kubernetes clusters from the command line.

#### Key Features
- **Automation:** Automates many tasks involved in cluster management, including installation, scaling, and upgrades.
- **AWS Integration:** Optimized for AWS but also supports GCE and other cloud providers to some extent.
- **Cluster Management:** Provides features for managing state, rolling updates, and managing the lifecycle of clusters.
- **Configuration:** Allows for a high degree of customization through configuration files.

#### Pros and Cons
- **Pros:** Good balance of automation and customization, production-ready, easier upgrades and management.
- **Cons:** Somewhat more complex than fully managed services like EKS, primarily designed for AWS.

### EKS (Elastic Kubernetes Service)

EKS is a managed Kubernetes service provided by AWS. It takes care of the heavy lifting involved in running Kubernetes clusters, such as infrastructure management, upgrades, and scaling.

#### Key Features
- **Managed Service:** AWS handles the availability and scalability of the Kubernetes control plane.
- **Integration with AWS Services:** Seamlessly integrates with other AWS services like IAM, VPC, CloudWatch, and more.
- **Security:** Provides built-in security features such as IAM roles for service accounts and fine-grained access control.
- **Maintenance:** AWS takes care of patching, updating, and managing the control plane components.

#### Pros and Cons
- **Pros:** Reduced operational overhead, high availability and scalability, deep integration with AWS services.
- **Cons:** Less control over the underlying infrastructure, potentially higher cost, dependent on AWS ecosystem.

### Detailed Comparison

| Feature                     | Kubeadm                                | Kops                                    | EKS                                      |
|-----------------------------|----------------------------------------|-----------------------------------------|------------------------------------------|
| **Control**                 | Full control over all components       | High control with some automation       | Limited control, AWS manages control plane |
| **Ease of Use**             | Requires Kubernetes expertise          | Easier than Kubeadm, but still technical| Easiest, managed by AWS                  |
| **Automation**              | Minimal automation                     | High level of automation                | Fully automated by AWS                   |
| **Maintenance**             | Manual updates and patches             | Automated but requires some intervention| Fully managed by AWS                     |
| **Cost**                    | Lower cost, only infrastructure        | Medium cost, depends on usage           | Higher cost, managed service premium     |
| **Flexibility**             | High flexibility for custom setups     | Flexible with customizable configurations| Less flexibility, follows AWS best practices |
| **Use Case**                | Custom infrastructure setups, learning | Production-grade clusters on AWS        | Easy, reliable clusters on AWS           |

### Choosing the Right Tool

- **Kubeadm** is best if you want to learn Kubernetes deeply, need a custom setup, or are running on on-premises infrastructure.
- **Kops** is a great choice for AWS users who need a production-ready solution with some level of control and automation.
- **EKS** is ideal if you prefer a managed service, want to reduce operational complexity, and are already invested in the AWS ecosystem.


![image](https://github.com/user-attachments/assets/0f2978dd-f2e8-46b8-93f1-d79036357d29)


EKS is a highly managed controlled plane but it is not managed data plane. It will give a higly available Kubernetes cluster with regard to the control plane.

To attach worker nodes we can create EC2 instances (we have to take care of high availablitiy of instances) or can use Fargate (AWS Serverless Compute that allows to run containers)

The prerequisites for this setup include:

1. kubectl: A command-line tool for managing Kubernetes clusters. It allows users to deploy, inspect, and manage Kubernetes resources such as pods, deployments, services, and more. Kubectl enables users to perform operations such as creating, updating, deleting, and scaling Kubernetes resources.

Run the following steps to install kubectl on your local machine / VM(EC2 instance - Ubuntu image).

```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```


Add execute permission to the binary:
```
chmod +x ./kubectl
```

Move the binary to a directory in your path:
```
sudo mv ./kubectl /usr/local/bin/kubectl
```

Verify the installation:

```
kubectl version --client
```

The output would look like this

![image](https://github.com/user-attachments/assets/223e9a45-66bf-4f20-8f7f-1fc439ba7797)

2. eksctl: A command-line tool designed for working with EKS clusters, streamlining various tasks. The eksctl tool uses CloudFormation under the hood, creating one stack for the EKS master control plane and another stack for the worker nodes.

Install and set up eksctl

Download and extract the latest release of eksctl with the following command.

```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp && sudo mv /tmp/eksctl /usr/local/bin
```

You can check the eksctl version using

```
eksctl version
```

The output would look like this

![image](https://github.com/user-attachments/assets/ea963b1c-4c7d-407a-b300-b930a44be52f)

3. AWS CLI: A command-line interface for interacting with AWS services, including Amazon EKS. To install, update, and uninstall AWS CLI, follow the instructions in the AWS Command Line Interface User Guide. After installation, it is advisable to configure the AWS CLI using the guidelines outlined in the AWS Command Line Interface User Guide under “Quick configuration with aws configure.”

Steps

Initially, access your AWS Console, log in, and execute the following command:

```
aws configure
```

![image](https://github.com/user-attachments/assets/a49c91f5-dedc-4650-a6ef-e9d03ce06d2e)

To get "Access Key ID" and "Seccret Access Key", log in to your AWS account and go to Security credentials.

![Screenshot from 2024-07-12 23-14-47](https://github.com/user-attachments/assets/a7aa04c9-f035-412e-9763-07ac927292f4)

Then go to Access keys section and create an access key:

![1 1](https://github.com/user-attachments/assets/bf8ab7a5-4a1a-4ab2-8c7a-3422df5dad1b)


### Create an EKS Cluster using EKSCTL

Now in this step, we are going to create Amazon EKS cluster using eksctl

```eksctl create cluster --name eksingressdemo --region us-east-1 --fargate```

--name eksingressdemo: Specifies the name of the EKS cluster to be created, in this case, "eksingressdemo".

--region us-east-1: Specifies the AWS region where the EKS cluster will be created, in this case, "us-east-1".

--fargate: Specifies that AWS Fargate should be used as the compute option for the EKS cluster.

AWS Fargate with Amazon EKS

AWS Fargate is a serverless compute engine for containers that works with both Amazon EKS and Amazon ECS. When you use Fargate, you don't need to provision or manage EC2 instances for running Kubernetes pods; instead, AWS manages the infrastructure for you.

Benefits of using Fargate with EKS include simplified cluster management, cost savings by paying only for the resources used by your pods, and improved scalability without worrying about underlying EC2 instances.

This will create an empty cluster, we will host the Kubernetes application on this cluster and see how Ingress Service caters requests from the client.

![1_6QEATFQfLzMOTB7z_k6lfQ](https://github.com/user-attachments/assets/c2d05617-0561-4dc7-aa27-93fb3af87795)

(Will take 10-20 minutes to create a cluster.)

Now go to EKS -> Clusters and click on the Cluster name

Here you can see in 'Overview' details of the created cluster.

![1 05](https://github.com/user-attachments/assets/c95a9171-8b74-40d5-beba-4b96190f55f8)

We can use 'Resources' tab to view what resources(pods, ReplicaSets, DaemonSets, Deployments, and other relevant components) are generated for the newly created cluster.

![1 2](https://github.com/user-attachments/assets/f8191d61-e9be-4067-9101-a437105b4bc3)

![1 3](https://github.com/user-attachments/assets/1fe61d6a-b8b5-4ae2-8893-70b9bc62c20b)

Under 'Compute' tab we can see nodes create using Fargate.

![1 4](https://github.com/user-attachments/assets/6a776209-8a43-4215-a939-0df904c5cae6)

If we have created node using EC2 instances manually, we can view them here. In this case I have used Fargate. And we can see the Fargate profile being created and we cann only create pods in the namespace available under that profile. (default and kube-system). We will later create a new namespace for our work.

![1 5](https://github.com/user-attachments/assets/d291156e-3c8f-4e9c-a596-84b6319c63d9)

Networking details,

![1 6](https://github.com/user-attachments/assets/f6b607d2-f28e-42e0-aae2-1b9718cd270f)

In Authentication, we can identity providers for cluster. But we are going to stick with IAM users identity as it is easy to use AWS resources together with Kubernetes clusters without any disturbances rather than using third party identity providers.

![1 7](https://github.com/user-attachments/assets/60beceef-b475-487f-9379-8e4d21251374)


![image](https://github.com/user-attachments/assets/1301de77-a085-4cba-886e-1232bca5026b)

You can also verify the status using console,

```
eksctl get cluster --name eksingressdemo --region us-east-1
```

This command retrieves the kubeconfig file. Rather than navigating through the resource tab and verifying on the AWS console, the kubectl command line can provide the same information.

```
aws eks update-kubeconfig --name test-cluster --region us-east-1
```

Next, we will create a Fargate profilewith the name “alb-sample-app.”

```
eksctl create fargateprofile \
    --cluster eksingressdemo \
    --region us-east-1 \
    --name alb-sample-app \
    --namespace game-2048
```

it creates a new Fargate profile together with the namespace "game-2048".

![1 8](https://github.com/user-attachments/assets/66dcf3c8-5518-46cd-a3d4-d594f775437a)

Then run the below command to apply file that contains all the configurations related deployment, service and ingress.

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
```

```
---
apiVersion: v1
kind: Namespace
metadata:
  name: game-2048
---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: game-2048
  name: deployment-2048
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: app-2048
  replicas: 5
  template:
    metadata:
      labels:
        app.kubernetes.io/name: app-2048
    spec:
      containers:
      - image: public.ecr.aws/l6m2t8p7/docker-2048:latest
        imagePullPolicy: Always
        name: app-2048
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  namespace: game-2048
  name: service-2048
spec:
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
  type: NodePort
  selector:
    app.kubernetes.io/name: app-2048
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  namespace: game-2048
  name: ingress-2048
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
spec:
  ingressClassName: alb
  rules:
    - http:
        paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: service-2048
              port:
                number: 80
```


Let’s break down the provided Kubernetes YAML manifest into different sections and explain each point:

```
apiVersion: v1
kind: Namespace
metadata:
  name: game-2048
```

* Purpose: Creates a Kubernetes namespace named game-2048 to isolate resources related to the 2048 game application.

#### Deployment

```
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: game-2048
  name: deployment-2048
spec:
  replicas: 5
  selector:
    matchLabels:
      app.kubernetes.io/name: app-2048
  template:
    metadata:
      labels:
        app.kubernetes.io/name: app-2048
    spec:
      containers:
        - name: app-2048
          image: public.ecr.aws/l6m2t8p7/docker-2048:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 80
```

* Purpose: Deploys the 2048 game application as a Kubernetes Deployment with 5 replicas.

-- template: Defines the pod template, including the container specifications.

-- containers: Specifies a container named app-2048 using the Docker image public.ecr.aws/l6m2t8p7/docker-2048:latest, exposing port 80 on the container.

-- selector: Ensures the Deployment manages pods with the label app.kubernetes.io/name: app-2048.


#### Service (NodePort)

```
apiVersion: v1
kind: Service
metadata:
  namespace: game-2048
  name: service-2048
spec:
  type: NodePort
  selector:
    app.kubernetes.io/name: app-2048
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
```

* Purpose: Creates a NodePort service named service-2048 to expose the Deployment internally within the cluster.

-- type: Specifies NodePort, which exposes the Service on a randomly selected port within the cluster's NodePort range (30000-32767 by default).

-- selector: Directs traffic to pods labeled app.kubernetes.io/name: app-2048. This should be same name as in spec -> template -> label in Deployment.yml

-- ports: Exposes port 80 on the Service and directs traffic to port 80 on the pods.


#### Ingress (ALB Ingress Controller) - Route the traffic inside the cluster

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  namespace: game-2048
  name: ingress-2048
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
spec:
  ingressClassName: alb
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: service-2048
                port:
                  number: 80
```

* Purpose: Configures an Ingress resource named ingress-2048 to expose the game application externally using an AWS ALB (Application Load Balancer) through the ALB Ingress Controller.

-- annotations: Specifies annotations for the ALB Ingress Controller:

-- alb.ingress.kubernetes.io/scheme: internet-facing: Configures the ALB to be internet-facing.

-- alb.ingress.kubernetes.io/target-type: ip: Specifies that the ALB should use IP addresses as targets.

-- ingressClassName: Specifies alb as the class name for the ALB Ingress Controller.

-- rules: Defines HTTP routing rules: Routes requests with path / (root path) to the service-2048 Service on port 80.


This YAML manifest sets up a complete environment for deploying and exposing the 2048 game application in Kubernetes:

**Namespace**: Provides isolation for resources related to the game application (game-2048).

**Deployment**: Manages multiple replicas of the game application using Docker containers.

**Service**: Exposes the game application internally within the Kubernetes cluster using NodePort.

**Ingress**: Exposes the game application externally via an AWS ALB, allowing access to the application through the specified URL path.

after that, you can see this kind of result;

![1 12](https://github.com/user-attachments/assets/6079d01f-26ab-4655-ab9e-70e7ef7a7685)


Subsequently, we can observe that all the pods are in a running state.

```
kubectl get pods -n game-2048
```

![1 9](https://github.com/user-attachments/assets/e1a1b176-e7ef-4dad-8f17-46ff80593e1f)

Upon inspecting the service, it becomes evident that it possesses only a node IP address. Those with VPC access can communicate with this pod using the any node IP address inside the VPC and port (31565). However, external users are unable to access it.

```
kubectl get svc -n game-2048
```

![1 10](https://github.com/user-attachments/assets/a985eb9e-f5e0-43dd-a1ca-7bfc296f88d2)

When reviewing the ingress resources, it’s apparent that an ingress is generated, but no address is assigned. The presence of an ingress controller is crucial. Once the ingress controller is deployed, the corresponding address will become visible.

```
kubectl get ingress -n game-2048
```

![1 11](https://github.com/user-attachments/assets/ae38a148-e790-413a-841c-49eca80b4dd6)


However, it’s important to note that we are solely generating resources for pod deployment, service, and ingress. The presence of a controller is essential for a comprehensive understanding of these resources.

Our next step involves creating an ingress controller. This controller will read the ingress resources within “ingress-2048,” configuring the entire load balancer. Within the Application Load Balancer (ALB), it will define the target group, ports, and other necessary configurations.


![1 12](https://github.com/user-attachments/assets/aaf883e9-9eed-4979-9711-f0f05993b933)
The running ALB controller requires access to AWS resources, necessitating integration with IAM (Identity and Access Management).
