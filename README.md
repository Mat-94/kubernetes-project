Kubernetes End-to-End Project: EKS Installation and Application
Deployment with Ingress
### Prerequisites
Before starting, ensure the following tools are installed and configured on your system:
1. kubectl: To interact with the Kubernetes cluster.
2. eksctl: A command-line tool to manage EKS clusters efficiently.
3. AWS CLI: To interact with AWS resources programmatically.
Ensure these tools are properly installed and authenticated with AWS credentials.
### Step 1: Create the EKS Cluster
We'll create the EKS cluster using `eksctl`, a preferred tool in many organizations for its automation
of manual configurations.
Run the following command to create the cluster with Fargate:
```bash
eksctl create cluster \
 --name my-new-cluster \
 --region us-east-1 \
 --fargate
```
This command automatically handles configurations that would otherwise be done manually through
the AWS console.
Once the cluster is created, you'll see confirmation similar to the screenshot below.
![Screenshot (33)](https://github.com/user-attachments/assets/071fbb75-5e31-4a3b-9f4a-c1319c534e1d)

![Screenshot (34)](https://github.com/user-attachments/assets/5140ba1a-f9d8-4232-869d-830cee89b16f)


### Step 2: Update kubectl Configuration
Update the `kubectl` context to interact with the newly created cluster:
```bash
aws eks update-kubeconfig \
 --name my-new-cluster \
 --region us-east-1
```
### Step 3: Create a Fargate Profile
Create a Fargate profile for application deployment. This profile defines namespaces for pods
running on Fargate:
```bash
eksctl create fargateprofile \
 --cluster my-new-cluster \
 --region us-east-1 \
 --name alb-sample-app \
 --namespace game-2048
```
The output will confirm the successful creation of the Fargate profile, including details about the
namespaces.
![Screenshot (35)](https://github.com/user-attachments/assets/cd09b71a-06f8-4a74-9b67-986c7f6e49af)



### Step 4: Deploy Application (Deployment, Service, and Ingress)
Deploy a sample gaming application along with its service and ingress configuration using the
provided YAML file:
```bash
kubectl apply -f
https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/exampl
es/2048/2048_full.yaml
```
This file can be viewed in a browser to understand its structure.
Ingress is used to route traffic within the cluster to the appropriate services.
### Step 5: Install the AWS Load Balancer Controller
Before installing the AWS Load Balancer (ALB) controller, create an IAM OIDC provider:
```bash
eksctl utils associate-iam-oidc-provider \
 --cluster my-new-cluster \
 --approve
```
This step integrates the IAM identity provider with the cluster.
### Step 6: Create an IAM Policy for the ALB Controller
Download the IAM policy:
```bash
curl -O
https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/i
am_policy.json
```
Create the policy in AWS:
```bash
aws iam create-policy \
 --policy-name AWSLoadBalancerControllerIAMPolicy \
 --policy-document file://iam_policy.json
```
### Step 7: Associate an IAM Service Account
Associate an IAM role with the ALB controller service account:
```bash
eksctl create iamserviceaccount \
 --cluster=my-new-cluster \
 --namespace=kube-system \
 --name=aws-load-balancer-controller \
 --role-name AmazonEKSLoadBalancerControllerRole \
--attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPoli
cy \
 --approve
```
### Step 8: Install the AWS Load Balancer Controller
1. Add the Helm repository:
 ```bash
 helm repo add eks https://aws.github.io/eks-charts
 ```
2. Update the repository:
 ```bash
 helm repo update eks
 ```
3. Install the ALB controller:
 ```bash
 helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
 -n kube-system \
 --set clusterName=my-new-cluster \
 --set serviceAccount.create=false \
 --set serviceAccount.name=aws-load-balancer-controller \
 --set region=us-east-1 \
 --set vpcId=vpc-00d5e88fcbe0493e6
 ```
### Step 9: Verify All Pods Are Running
Check if all the pods in the `kube-system` namespace are running:
```bash
kubectl get pods -n kube-system
```
### Step 10: Access the Application
1. Navigate to the **Load Balancer** section in the AWS Management Console.
2. Locate the DNS name of the ALB created for your application.
3. Copy the DNS name and paste it into your browser's address bar.
You should see the deployed game application interface.
![IMG_2105](https://github.com/user-attachments/assets/d8e7e047-b032-4b13-8c1c-8aefb6af065e)

### Project Outcome
This project demonstrates the following:
- Setting up an Amazon EKS cluster using `eksctl`.
- Configuring and deploying a sample application on Kubernetes with Fargate.
- Integrating AWS ALB for traffic management within the cluster.
- Accessing the deployed application through an ingress controller.
This comprehensive workflow highlights your proficiency in Kubernetes and AWS, making it a
valuable addition to your professional portfolio.
