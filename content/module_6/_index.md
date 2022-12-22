+++
title = "Module 6 - Configure Application Load Balancer (ALB)"
weight = 60
+++

For customers looking for an AWS-native approach for load balancing at the edge, the AWS Load Balancer Controller manages AWS Elastic Load Balancers for a Kubernetes cluster and provisions an AWS Application Load Balancer (ALB) when you create a Kubernetes Ingress. To deploy this solution, visit [Installing the AWS Load Balancer Controller add-on](https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html).

**Step 1**: Download an IAM policy for the AWS Load Balancer Controller that allows it to make calls to AWS APIs on your behalf.
```
    curl -o iam_policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.4.4/docs/install/iam_policy.json
```

**Step 2**: Create an IAM policy using the policy downloaded in the previous step. 
```
    ALB_IAM_POLICY=$(aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json --query Policy.PolicyArn --text) && echo 'ALB_IAM_POLICY='$ALB_IAM_POLICY
```
    
**Step 3:** Create an IAM Role
Create a Kubernetes service account named `aws-load-balancer-controller` in the `kube-system` namespace for the AWS Load Balancer Controller and annotate the Kubernetes service account with the name of the IAM role.

You can use eksctl or the AWS CLI and kubectl to create the IAM role and Kubernetes service account.
```
    eksctl create iamserviceaccount \
      --cluster=$eks_cluster_name  \
      --namespace=kube-system \
      --name=aws-load-balancer-controller \
      --role-name "AmazonEKSLoadBalancerControllerRole" \
      --attach-policy-arn=$ALB_IAM_POLICY \
      --approve
```
  
**Step 4:** Install the AWS Load Balancer Controller using Helm 
```
    helm repo add eks https://aws.github.io/eks-charts
    helm repo update
    helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
      -n kube-system \
      --set clusterName=$eks_cluster_name \
      --set serviceAccount.create=false \
      --set serviceAccount.name=aws-load-balancer-controller 
```

**Step 5:** Patch aws-load-balancer-controller Deployment to Support Multiple Wavelength Zones
```
    kubectl patch deployment aws-load-balancer-controller -n kube-system -p '{"spec": {"template": {"spec": {"nodeSelector": {"topology": "us-west-2a"}}}}}'
```

### Create Sample Workload
In this example, we will create a simple web Deployment in front of an Ingress, which will provision an ALB upon creation.