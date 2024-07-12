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

