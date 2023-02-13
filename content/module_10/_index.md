+++
title = "Module 10 - Deleting AWS Wavelength Resources"
weight = 90
+++

In this last step, you will clean-up the resources created from this wokrshop the running application. 

* Terminate the two running Amazon EC2 instances from Module 2 (Wavelength Zone Instance) and Module 4 (Bastion Host).
    ```
    aws ec2 terminate-instances --instance-ids $BASTION_INSTANCE_ID $EC2_WLZ_ID
    ```
    
* Next, disassociate the Carrier IP addresses from the EKS Worker Node, delete the self-managed node CloudFormation Stack, and delete the EKS cluster.
    ```
    aws ec2 disassociate-address --association-id $AssociationId
    aws ec2 release-address --network-border-group us-west-2-wl1-las-wlz-1 --allocation-id $eks_allocation_id
    aws cloudformation delete-stack --stack-name $stack_name
    aws eks delete-cluster --name $eks_cluster_name
    aws iam detach-role-policy --role-name $eks_role --policy-arn $aws_eks_policy 
    aws iam delete-role --role-name $eks_role
    aws ec2 delete-security-group --group-id $EKS_CLUSTER_SG_ID
    ```

* Navigate to the [VPC Management Console](https://us-west-2.console.aws.amazon.com/vpc/home?region=us-west-2#vpcs:) to see the list of your running VPCs. 
* When you find the VPC named **Wavelength-Workshop-VPC**, select the checkbox on the left-hand side, then select **Actions** in the top right followed by **Delete VPC**.
* To confirm deletion, follow the instructions and type **delete** in the user input field, followed by the **Delete** button in orange.


