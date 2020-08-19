+++
title = "Step 4 - Create Elastic IPs and Network Interfaces"
weight = 50
+++

The final step before deploying the actual instances is to create two
Elastic IPs which will come from the carrier network. These IP
addresses will be assigned to two Elastic Network Interfaces (ENIs), and
the ENIs will be assigned to our API and Inference server (the bastion
host will have it's public IP assigned upon creation by the bastion
subnet).

##### Create the Elastic IPs

1.  Create an EIP for the API server. This IP will be on the carrier network. 

        export API_CIP_ALLOC_ID=$(aws ec2 allocate-address \
        --region $REGION \
        --output text \
        --domain vpc \
        --network-border-group $NBG \
        --query 'AllocationId') \
        && echo '\nAPI_CIP_ALLOC_ID='$API_CIP_ALLOC_ID

1. Take note of the API server carrier (public) IP address.

        aws ec2 describe-addresses --allocation-ids $API_CIP_ALLOC_ID \
        --query 'Addresses[*].{"API server carrier (public) IP":CarrierIp}'

1.  Create an Elastic IP (EIP) for the inference server. This IP will be on the carrier network. 

        export INFERENCE_CIP_ALLOC_ID=$(aws ec2 allocate-address \
        --region $REGION \
        --output text \
        --domain vpc \
        --network-border-group $NBG \
        --query 'AllocationId') \
        && echo '\nINFERENCE_CIP_ALLOC_ID='$INFERENCE_CIP_ALLOC_ID

##### Create the Elastic Network Interfaces

2.  Create an Elastic Network Interface (ENI) for the inference server. 

        export INFERENCE_ENI_ID=$(aws ec2 create-network-interface \
        --region $REGION \
        --output text \
        --subnet-id $WL_SUBNET_ID \
        --groups $INFERENCE_SG_ID \
        --query 'NetworkInterface.NetworkInterfaceId') \
        && echo '\nINFERENCE_ENI_ID='$INFERENCE_ENI_ID

2.  Create an ENI for the API server. 

        export API_ENI_ID=$(aws ec2 create-network-interface \
        --region $REGION \
        --output text \
        --subnet-id $WL_SUBNET_ID \
        --groups $API_SG_ID \
        --query 'NetworkInterface.NetworkInterfaceId') \
        && echo '\nAPI_ENI_ID='$API_ENI_ID

##### Assciate the EIPs with the ENIs

3.  Associate the inference EIP with the ENI

        aws ec2 --region $REGION associate-address \
        --allocation-id $INFERENCE_CIP_ALLOC_ID \
        --network-interface-id $INFERENCE_ENI_ID

3.  Associate the API EIP with the ENI

        aws ec2 --region $REGION associate-address \
        --allocation-id $API_CIP_ALLOC_ID \
        --network-interface-id $API_ENI_ID