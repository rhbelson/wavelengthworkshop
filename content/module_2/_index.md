+++
title = "Module 2 - Create EC2 Instance with Carrier IP"
weight = 30
+++

In this section you'll add three security groups:

-   **Bastion SG** allows SSH traffic from your local machine to the
    bastion host as well as HTTP web traffic from the Internet

-   **API SG** allows SSH traffic from the Bastion SG and opens up port
    5000 to accept incoming API requests

-   **Inference SG** allows SSH traffic from the Bastion host and
    communications on port 8080 and 8081 (the ports used by the
    inference server) from the API SG.

##### Create the Bastion security group

*  Create the bastion security group.

        export BASTION_SG_ID=$(aws ec2 create-security-group \
        --region $REGION \
        --output text \
        --group-name bastion-sg \
        --description "Security group for bastion host" \
        --vpc-id $VPC_ID \
        --query 'GroupId') \
        && echo '\nBASTION_SG_ID='$BASTION_SG_ID

* Create an ingress rule to allow SSH from your current IP address. If you want to access SSH from anywhere change `$(curl https://checkip.amazonaws.com)/32` to `0.0.0.0/0` - but be aware that this allows access from any IP address and is considered a security anti-pattern. 

        aws ec2 authorize-security-group-ingress \
        --region $REGION \
        --group-id $BASTION_SG_ID \
        --protocol tcp \
        --port 22 \
        --cidr $(curl https://checkip.amazonaws.com)/32

* Create an ingress rule to all web traffic from any IP address. 

        aws ec2 authorize-security-group-ingress \
        --region $REGION \
        --group-id $BASTION_SG_ID \
        --protocol tcp \
        --port 80 \
        --cidr 0.0.0.0/0

##### Create the API security group

*  Create the API security group.

        export API_SG_ID=$(aws ec2 create-security-group \
        --region $REGION \
        --output text \
        --group-name api-sg \
        --description "Security group for API host" \
        --vpc-id $VPC_ID \
        --query 'GroupId') \
        && echo '\nAPI_SG_ID='$API_SG_ID

* Create an ingress rule that allows SSH from the Bastion security group

        aws ec2 authorize-security-group-ingress \
        --region $REGION \
        --group-id $API_SG_ID \
        --protocol tcp \
        --port 22 \
        --source-group $BASTION_SG_ID

* Create an ingress rule that allows API requests over port 5000 from the Internet. 

        aws ec2 authorize-security-group-ingress \
        --region $REGION \
        --group-id $API_SG_ID \
        --protocol tcp \
        --port 5000 \
        --cidr 0.0.0.0/0

##### Create the Inference security group

*  Create the security group for the inference server.export 

        INFERENCE_SG_ID=$(aws ec2 create-security-group \
        --region $REGION \
        --output text \
        --group-name inference-sg \
        --description "Security group for inference host" \
        --vpc-id $VPC_ID \
        --query 'GroupId') \
        && echo '\nINFERENCE_SG_ID='$INFERENCE_SG_ID

* Create an ingress rule that allows SSH from the Bastion security group

        aws ec2 authorize-security-group-ingress \
        --region $REGION \
        --group-id $INFERENCE_SG_ID \
        --protocol tcp \
        --port 22 \
        --source-group $BASTION_SG_ID

* Create an ingress rule to allow infeference processing traffic from the API server.

        aws ec2 authorize-security-group-ingress \
        --region $REGION \
        --group-id $INFERENCE_SG_ID \
        --protocol tcp \
        --port 8080 \
        --source-group $API_SG_ID

* Create an ingress rule to allow management api requests from the API server.

        aws ec2 authorize-security-group-ingress \
        --region $REGION \
        --group-id $INFERENCE_SG_ID \
        --protocol tcp \
        --port 8081 \
        --source-group $API_SG_ID
