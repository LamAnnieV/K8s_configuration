# How to Configure EKS

In the Jenkins Agent - Kubernetes:

### Create the EKS Cluster

```
eksctl create cluster cluster01 \
  --vpc-private-subnets="<enter private subnet ids separated by commas with no space>" \
  --vpc-public-subnets="<enter private subnet ids separated by commas with no space>" \
  --without-nodegroup
```


### To Create one node in each private subnet:

```
eksctl create nodegroup --cluster cluster01 --node-private-networking --node-type t2.medium --nodes 3
```


### Provided permission to my AWS account to interact with our cluster through *OpenID*

`eksctl utils associate-iam-oidc-provider --cluster cluster01 --approve
`aws iam list-open-id-connect-providers


###  Tag the public subnets in the Application Infrastructure:

Add the following tags to each of the public subnets  "kubernetes.io/role/elb" = "1"


### Downloaded iampolicy.json file in our EKS cluster to define what is allowable and what isn’t when accessing our application. 

```
wget https://raw.githubusercontent.com/kura-labs-org/Template/main/iam_policy.json
```


### Downloaded “AWSLoadBalancerControllerIAMPolicy" to give permission to the cluster to use my ALB controller.

`aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json


### Create a service account that is used to control access to our ingress controller to interact with our ALB in our EKS cluster

```
eksctl create iamserviceaccount \
--cluster=cluster01 \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::848991144892:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--approve
```


### Created certificate manager to secure the traffic from clients and associated ingress (incoming traffic) controller with the associated domain name that will be created to access our application through the load balancer
```
kubectl apply \
--validate=false \
-f https://github.com/jetstack/cert-manager/releases/download/v1.5.4/cert-manager.yaml
```


### List of policies under the AWSLoadBalancerControllerRole on our worker nodes (2 instances configured to spin back up if ever terminated:

### Configured ALB controller:

•	Downloaded v2_4_5_full.yaml file from GitHub: 

```wget https://github.com/kubernetes-sigs/aws-load-balancer-controller/releases/download/v2.4.5/v2_4_5_full.yaml``` which acts as our *IngressClass.yaml* file to define the ingress controller we’re creating to handle the ALB controller resource to balance traffic load. *changed cluster name to our EKS cluster*

### Update the Kubernetes configuration (kubeconfig) file for the specified Amazon EKS (Elastic Kubernetes Service) cluster named "cluster01" in the "us-east-1" region. 

`aws eks --region us-east-1 update-kubeconfig --name cluster01

### Apply the Custom Resource Definitions (CRDs) for Cert-Manager

kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.6.1/cert-manager.crds.yaml

### Configured ALB controller:

kubectl get deployment -n kube-system aws-load-balancer-controller

kubectl apply -k "github.com/aws/eks-charts/stable/aws-load-balancer-controller/crds"







