+++
title = "Step 10 - Summary & Cleanup"
weight = 90
+++

In this last step, you will clean-up the resources created from this wokrshop the running application. 

* Terminate the 3 running Amazon EC2 instances
    ```
    aws ec2 terminate-instances --instance-ids $API_INSTANCE_ID $BASTION_INSTANCE_ID $INFERENCE_INSTANCE_ID
    ```
    
* Next, disassociate the Carrier IP addresses from the API Server and Inference Server
    ```
    aws ec2 disassociate-address --association-id $API_ASSOCIATION_ID
    aws ec2 disassociate-address --association-id $INF_ASSOCIATION_ID
    ```

* Next, release the Carrier IP addresses that were allocated to the API Server and Inference Server
    ```
    aws ec2 release-address --network-border-group ca-central-1-wl1-yto-wlz-1 --allocation-id $API_CIP_ALLOC_ID
    aws ec2 release-address --network-border-group ca-central-1-wl1-yto-wlz-1 --allocation-id $INFERENCE_CIP_ALLOC_ID
    ```
    
* Finally, delite the Elastic Network Interfaces (ENI) created for the two EC2 instances
    ```
    aws ec2 delete-network-interface --network-interface-id $INFERENCE_ENI_ID
    aws ec2 delete-network-interface --network-interface-id $API_ENI_ID
    ```

* Navigate to the [VPC Managent Console](https://ca-central-1.console.aws.amazon.com/vpc/home?region=ca-central-1#vpcs:) to see the list of your running VPCs. 
* When you find the VPC named **Bell-5G-Wavelength-VPC**, select the checkbox on the left-hand side, then select **Actions** in the top right followed by **Delete VPC**.
* To confirm deletion, follow the instructions and type **delete** in the user input field, followed by the **Delete** button in orange.


