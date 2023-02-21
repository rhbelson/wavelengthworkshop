+++
title = "Step 1 - Deploy the VPC"
weight = 20
+++

#### The first step in this tutorial will be to deploy the VPC, Internet gateway, and carrier gateway.

*  First, we need to confirm vCPU limits to ensure your account can launch two t3.medium instances and one g4dn.2xlarge instances.
    ```
    aws service-quotas request-service-quota-increase --service-code ec2 --quota-code L-DB2E81BA --region eu-west-2 --desired-value 16
    aws service-quotas request-service-quota-increase --service-code ec2 --quota-code L-1216C47A --region eu-west-2 --desired-value 16
    ```

*  In order to get started, you will set some environment variables. Start by generating a key pair to use in the region (defaults to eu-west-2). If you have not done so, please run the following:

    ```
    aws ec2 create-key-pair --key-name my_key_pair --region eu-west-2 --query 'KeyMaterial' --output text > my_key_pair.pem
    chmod 400 my_key_pair.pem
    ```
    
*  Next, let's define the environment variables that we'll use throughout the workshop.
    ```
    export REGION="eu-west-2"
    export WL_ZONE="eu-west-2-wl1-man-wlz-1"
    export NBG="eu-west-2-wl1-man-wlz-1"
    export INFERENCE_IMAGE_ID="ami-0d98aa2705c5e7693"
    export API_IMAGE_ID="ami-07092d732afc011fa"
    export BASTION_IMAGE_ID="ami-08cd358d745620807"
    export KEY_NAME="my_key_pair"
    ```

*  Use the AWS CLI to create the VPC
    ```
    export VPC_ID=$(aws ec2 create-vpc  \
    --region $REGION  \
    --output text  \
    --cidr-block 10.0.0.0/16  \
    --tag-specifications ResourceType=vpc,Tags='[{Key=Name,Value="Vodafone-Distributed-MEC-VPC"}]' \
    --query 'Vpc.VpcId')  \
    && echo '\nVPC_ID='$VPC_ID
    ```
*  Create an Internet Gateway 

        export IGW_ID=$(aws ec2 create-internet-gateway  \
        --region $REGION  \
        --output text  \
        --query 'InternetGateway.InternetGatewayId')  \
        && echo ' \nIGW_ID='$IGW_ID

* Attach the Internet Gateway to the VPC

        aws ec2 attach-internet-gateway  \
        --region $REGION  \
        --vpc-id $VPC_ID  \
        --internet-gateway-id $IGW_ID

*  Add the Carrier Gateway

        export CAGW_ID=$(aws ec2 create-carrier-gateway  \
        --region $REGION  \
        --output text  \
        --vpc-id $VPC_ID  \
        --query 'CarrierGateway.CarrierGatewayId')  \
        && echo '\nCAGW_ID='$CAGW_ID
