+++
title = "Module 5 - Create EKS Cluster"
weight = 60
+++

In this module, we will extend our existing VPC to include an EKS Cluster, consisting of the control plane in the AWS Region and self-managed nodes in the AWS Wavelength Zone. You can learn more about best practices for Amazon EKS with AWS Wavelength with this blog: [Deploy geo-distributed Amazon EKS clusters on AWS Wavelength](https://aws.amazon.com/blogs/containers/deploy-geo-distributed-amazon-eks-clusters-on-aws-wavelength/).

##### Create the Security Groups 
In this section, we will create an additional security group for our EKS control plane. This will ensure least-privilege access to our internal EKS Cluster endpoints.

**Step 1:** Create the Security Group to be used for EKS Cluster
```
        EKS_CLUSTER_SG_ID=$(aws ec2 create-security-group --group-name wl-eks-cluster-sg --description "wl-eks-cluster-sg"  --vpc-id $VPC_ID --tag-specifications 'ResourceType=security-group,Tags=[{Key=Name,Value=wl-eks-cluster-sg}]' --query 'GroupId' --output text)
        aws ec2 describe-security-groups --group-ids $EKS_CLUSTER_SG_ID
        
        Sample Output:
        {
            "SecurityGroups": [
                {
                    "Description": "wl-eks-cluster-sg",
                    "GroupName": "wl-eks-cluster-sg",
                    "IpPermissions": [],
                    "OwnerId": "454545454545",
                    "GroupId": "sg-0d9ff6f450281c814",
                    "IpPermissionsEgress": [
                        {
                            "IpProtocol": "-1",
                            "IpRanges": [
                                {
                                    "CidrIp": "0.0.0.0/0"
                                }
                            ],
                            "Ipv6Ranges": [],
                            "PrefixListIds": [],
                            "UserIdGroupPairs": []
                        }
                    ],
                    "Tags": [
                        {
                            "Key": "Name",
                            "Value": "wl-eks-cluster-sg"
                        }
                    ],
                    "VpcId": "vpc-01c2ddabfd16b92f1"
                }
            ]
        }
```

**Step 2:** Allow HTTPS for the EKS Security Group
```
        aws ec2 authorize-security-group-ingress --group-id $EKS_CLUSTER_SG_ID --ip-permissions '[{"IpProtocol": "tcp", "FromPort": 443, "ToPort": 443, "IpRanges": [{"CidrIp": "0.0.0.0/0", "Description": "HTTPS"}]}]'
        
        aws ec2 describe-security-groups --group-ids $EKS_CLUSTER_SG_ID
        
        Sample Output:
        {
            "SecurityGroups": [
                {
                    "Description": "wl-eks-cluster-sg",
                    "GroupName": "wl-eks-cluster-sg",
                    "IpPermissions": [
                        {
                            "FromPort": 443,
                            "IpProtocol": "tcp",
                            "IpRanges": [
                                {
                                    "CidrIp": "0.0.0.0/0",
                                    "Description": "HTTPS"
                                }
                            ],
                            "Ipv6Ranges": [],
                            "PrefixListIds": [],
                            "ToPort": 443,
                            "UserIdGroupPairs": []
                        }
                    ],
                    "OwnerId": "454545454545",
                    "GroupId": "sg-0d9ff6f450281c814",
                    "IpPermissionsEgress": [
                        {
                            "IpProtocol": "-1",
                            "IpRanges": [
                                {
                                    "CidrIp": "0.0.0.0/0"
                                }
                            ],
                            "Ipv6Ranges": [],
                            "PrefixListIds": [],
                            "UserIdGroupPairs": []
                        }
                    ],
                    "Tags": [
                        {
                            "Key": "Name",
                            "Value": "wl-eks-cluster-sg"
                        }
                    ],
                    "VpcId": "vpc-01c2ddabfd16b92f1"
                }
            ]
        }
```

##### Create IAM Roles for EKS Cluster


**Step 1:** Create the Role Policy Document
```
        cat > eks-role.json <<EOF
        {
          "Version": "2012-10-17",
          "Statement": [
            {
              "Effect": "Allow",
              "Principal": {
                "Service": "eks.amazonaws.com"
              },
              "Action": "sts:AssumeRole"
            }
          ]
        }
        EOF
```

**Step 2:** Create a IAM role (to be used for EKS Cluster Creation)
```
        export eks_role="wavelength-eks-role"
        
        aws iam create-role --role-name $eks_role --assume-role-policy-document file://./eks-role.json
        
        Sample Output:
        {
            "Role": {
                "Path": "/",
                "RoleName": "wavelength-eks-role",
                "RoleId": "AROASV6M4JDVYB6H562OJ",
                "Arn": "arn:aws:iam::454545454545:role/wavelength-eks-role",
                "CreateDate": "2020-10-11T02:53:59+00:00",
                "AssumeRolePolicyDocument": {
                    "Version": "2012-10-17",
                    "Statement": [
                        {
                            "Effect": "Allow",
                            "Principal": {
                                "Service": "eks.amazonaws.com"
                            },
                            "Action": "sts:AssumeRole"
                        }
                    ]
                }
            }
        }
```

**Step 3:** Attach the EKS Service Policy to the Created Role

```
        export aws_eks_policy="arn:aws:iam::aws:policy/AmazonEKSClusterPolicy"

        aws iam attach-role-policy --role-name $eks_role --policy-arn $aws_eks_policy 
```

**Step 4:** Set the Cluster Name and role to be attached

```
        export eks_cluster_name="wavelength_eks"

        eks_aws_role_arn=$(aws iam get-role --role-name $eks_role --query 'Role.Arn' --output text)
```


#### Create the EKS Cluster
Now we are ready to launch the EKS Control Plane! To do so, we will use the `create-cluster` command, passing in a number of attributes including the Kubernetes Version, IAM Role, subnets for our control plane, cluster security group, and cluster endpoint access configuration.

```
        aws eks create-cluster --name $eks_cluster_name  --kubernetes-version 1.21 --role-arn $eks_aws_role_arn --resources-vpc-config subnetIds=$BASTION_SUBNET_ID,$BASTION_SUBNET_ID2,securityGroupIds=$EKS_CLUSTER_SG_ID,endpointPublicAccess=true,endpointPrivateAccess=true
        
        Sample Output:
        {
            "cluster": {
                "name": "wavelength_eks",
                "arn": "arn:aws:eks:us-west-2:963276066795:cluster/wavelength_eks",
                "createdAt": "2021-11-29T18:40:56.223000-08:00",
                "version": "1.21",
                "roleArn": "arn:aws:iam::963276066795:role/wavelength-eks-role",
                "resourcesVpcConfig": {
                    "subnetIds": [
                        "subnet-048faba6216af6f0f",
                        "subnet-0c94043f88bde9207"
                    ],
                    "securityGroupIds": [
                        "sg-031a9d9f92e4d6d3f"
                    ],
                    "vpcId": "vpc-0ec6a8d4f38771285",
                    "endpointPublicAccess": true,
                    "endpointPrivateAccess": true,
                    "publicAccessCidrs": [
                        "0.0.0.0/0"
                    ]
                },
                "kubernetesNetworkConfig": {
                    "serviceIpv4Cidr": "172.20.0.0/16"
                },
                "logging": {
                    "clusterLogging": [
                        {
                            "types": [
                                "api",
                                "audit",
                                "authenticator",
                                "controllerManager",
                                "scheduler"
                            ],
                            "enabled": false
                        }
                    ]
                },
                "status": "CREATING",
                "certificateAuthority": {},
                "platformVersion": "eks.3",
                "tags": {}
            }
        }
```

After creating the cluster, it might take 10-15 minutes to complete the requisite steps for your cluster to be active. To check the latest EKS cluster status, run the following:

```
        aws eks describe-cluster --name $eks_cluster_name --query 'cluster.status' --output text

        Sample Output (before ~10min elapses):
        CREATING 
```

While you cannot configure the cluster until the status is ACTIVE, let's proceed to configuring the worker nodes, while we wait for the cluster to spin up.


#### Configure Worker Nodes
In this section, we will need to create a self-managed node group for the Wavelength Zone, as managed nodes are not supported. To do so, we will leveraged a well-defined CloudFormation template to configure the needed security groups, IAM roles, and kubelet configs for this AWS Auto Scaling Group.

**Step 1:** Check the EKS Cluster status (Do Not Proceed Unless Output shows `Active`)
```
        aws eks describe-cluster --name $eks_cluster_name --query 'cluster.status' --output text
        
        Sample Output:
        ACTIVE
```

**Step 2:** Describe the EKS Cluster to obtain the API Server Endpoint and Cluster Certificate
```
        api_server=$(aws eks describe-cluster --name $eks_cluster_name --query 'cluster.endpoint' --output text)
        
        certificate_authority=$(aws eks describe-cluster --name $eks_cluster_name --query 'cluster.certificateAuthority.data' --output text)
        
        echo $api_server $certificate_authority
```

**Step 3:** Execute the CloudForamtion Stack to create Self-Managed Worker Node 
In this step, we'll use the two variables (api_server,certificate_authority) above as parameters (specifically for the BootstrapArguments to the EKS Cluster) to the CloudFormation template attached.
```
        export nodegroup_name="wavelength-eks-nodegroup"
        
        export stack_name="wavelength-eks-node"
        
        aws cloudformation create-stack --stack-name $stack_name --template-url https://amazon-eks.s3.us-west-2.amazonaws.com/cloudformation/2020-08-12/amazon-eks-nodegroup.yaml --parameters ParameterKey=ClusterControlPlaneSecurityGroup,ParameterValue=$EKS_CLUSTER_SG_ID ParameterKey=NodeGroupName,ParameterValue=$nodegroup_name ParameterKey=KeyName,ParameterValue=test_key_uswest2 ParameterKey=Subnets,ParameterValue=$LAS_WLZ_SUBNET ParameterKey=VpcId,ParameterValue=$VPC_ID ParameterKey=ClusterName,ParameterValue=$eks_cluster_name ParameterKey=NodeAutoScalingGroupDesiredCapacity,ParameterValue=1 ParameterKey=BootstrapArguments,ParameterValue="--apiserver-endpoint $api_server  --b64-cluster-ca $certificate_authority" --capabilities CAPABILITY_NAMED_IAM
        
        Sample Output:
        {
            "StackId": "arn:aws:cloudformation:us-east-1:454545454545:stack/wavelength-eks-node/439165a0-0b55-11eb-8925-12e346dd5d0c"
        }
```

**Step 4:** Check the Status of the Stack 
Please wait ~5min for the CloudFormation Stack to finish provisioning resources. Do not proceed ot the next step until the output reads `CREATE_COMPLETE` and not `CREATE_IN_PROGRESS`.
```
        aws cloudformation describe-stacks --stack-name $stack_name --query 'Stacks[].StackStatus' --output text
        
        Sample Output:
        CREATE_COMPLETE
```

#### Configure EKS Cluster to Register Self-Managed Nodes
We're almost ready to deploy a sample workload to our EKS Cluster! To grant additional AWS users or roles -- such as the IAM role attached to the self-managed nodes –– the ability to interact with your cluster, you must edit the aws-auth ConfigMap within Kubernetes and create a Kubernetes rolebinding or clusterrolebinding with the name of a group that you specify in the `aws-auth` ConfigMap. 

**Step 1:** Obtain the EKS Node Profile Variable and Security Group of the Self-Managed Nodes
```
eks_node_profile=$(aws cloudformation describe-stacks --stack-name $stack_name --query "Stacks[0].Outputs[?OutputKey=='NodeInstanceRole'].OutputValue" --output text) && echo 'eks_node_profile='$eks_node_profile
eks_node_securitygroup=$(aws cloudformation describe-stacks --stack-name $stack_name --query "Stacks[0].Outputs[?OutputKey=='NodeSecurityGroup'].OutputValue" --output text) && echo 'eks_node_securitygroup='$eks_node_securitygroup
```

**Step 2:** Allocate a Carrier IP for the EKS Worker Node
In this step, we will demonstrate how to provide your worker nodes with access to the carrier network and public internet. Please note, however, the following caveats:
- This method supports a single Carrier IP attached to a single self-managed node. To support multiple nodes, you will need to allocate/attach as many Carrier IPs as there are worker nodes
- This method enables Kubernetes services to be exposed via [NodePorts](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport), which is not recommended at scale
- This method positions an alternative to the Application Load Balancer, discussed in the next module

```
        #Retrieve the Auto Scaling Group for the EKS Cluster
        eks_node_scalinggroup=$(aws cloudformation describe-stacks --stack-name $stack_name --query "Stacks[0].Outputs[?OutputKey=='NodeAutoScalingGroup'].OutputValue" --output text)
        echo $eks_node_scalinggroup
        
        #Retrieve the Instance ID of the Instance in the Nodegroup
        eks_node1_id=$(aws autoscaling describe-auto-scaling-groups --auto-scaling-group-names $eks_node_scalinggroup  --query "AutoScalingGroups[0].Instances[0].InstanceId" --output text)
        echo $eks_node1_id
        
        #Determining the NIC ID of the Instance in the Nodegroup
        eks_node1_nic_id=$(aws ec2 describe-instances --instance-id $eks_node1_id  --query "Reservations[0].Instances[0].NetworkInterfaces[0].NetworkInterfaceId" --output text)
        echo $eks_node1_nic_id
        
        #Allocate a Carrier IP for the EKS Worker Node
        LAS_WLZ_ID=us-west-2-wl1-las-wlz-1
        eks_allocation_id=$(aws ec2 allocate-address --network-border-group $LAS_WLZ_ID --query AllocationId --output text)
        
        #Associate the Carrier IP to the EKS worker node network inferface
        AssociationId=$(aws ec2 associate-address --allocation-id $eks_allocation_id --network-interface-id $eks_node1_nic_id --query 'AssociationId' --output text)
        echo $AssociationId
```

**Step 3:** Test Cluster Endpoint Access
In this step, we will quickly update our local kubeconfig to match the EKS Cluster we created. If successful, running `kubectl get svc` should return the following:

```
        aws eks update-kubeconfig --name $eks_cluster_name 
        kubectl get svc
        
        Sample Output:
        
        NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
        kubernetes   ClusterIP   172.20.0.1   <none>        443/TCP   24m
```

**Step 4:** Check the aws-auth ConfigMap 
By default, no aws-auth Config map should be found, but let's check anyways.

```
        kubectl describe configmap -n kube-system aws-auth
        
        Sample Output:
        Error from server (NotFound): configmaps "aws-auth" not found
```

**Step 5:** Create & Apply the aws-auth ConfigMap
```
        cat > aws-auth-cm.yaml <<EOF
        apiVersion: v1
        kind: ConfigMap
        metadata:
          name: aws-auth
          namespace: kube-system
        data:
          mapRoles: |
            - rolearn: $eks_node_profile
              username: system:node:{{EC2PrivateDNSName}}
              groups:
                - system:bootstrappers
                - system:nodes
        EOF
        
        kubectl apply -f aws-auth-cm.yaml
        
        Sample Output:
        configmap/aws-auth created
```

**Step 6:** Confirm EKS Cluster nodes
```
        kubectl get nodes
        
        Sample Output (will take several seconds for the node to become Ready):
        
        NAME                         STATUS   ROLES    AGE     VERSION
        ip-10-0-1-235.ec2.internal   Ready    <none>   7h52m   v1.17.11-eks-cfdc40
```

Congratulations! We have completed setting up our EKS Cluster and worker nodes.