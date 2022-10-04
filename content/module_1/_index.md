+++
title = "Step 1 - Deploy the VPC"
weight = 20
+++

#### The first step in this tutorial will be to deploy the VPC, Internet gateway, and carrier gateway.



*  In order to get started, you will set some environment variables.
    
    ***Note***: replace the value for KEY_NAME with the name of the key pair
    you wish to use. 
    
    ***Note***: Choose the set of environment variables for the approprite Region.

        export REGION="ca-central-1"
        export WL_ZONE="ca-central-1-wl1-yto-wlz-1"
        export NBG="ca-central-1-wl1-yto-wlz-1"
        export INFERENCE_IMAGE_ID="ami-014bc836b5c3e330e"
        export API_IMAGE_ID="ami-089f7c9524e7245f9"
        export BASTION_IMAGE_ID="ami-0a70476e631caa6d3"

        export KEY_NAME=<your key name>   

*  Use the AWS CLI to create the VPC

        export VPC_ID=$(aws ec2 create-vpc  \
        --region $REGION  \
        --output text  \
        --cidr-block 10.0.0.0/16  \
        --query 'Vpc.VpcId')  \
        && echo '\nVPC_ID='$VPC_ID

*  Create an Internet gateway 

        export IGW_ID=$(aws ec2 create-internet-gateway  \
        --region $REGION  \
        --output text  \
        --query 'InternetGateway.InternetGatewayId')  \
        && echo ' \nIGW_ID='$IGW_ID

* Attach the Internet gateway to the VPC

        aws ec2 attach-internet-gateway  \
        --region $REGION  \
        --vpc-id $VPC_ID  \
        --internet-gateway-id $IGW_ID

*  Add the carrier gateway

        export CAGW_ID=$(aws ec2 create-carrier-gateway  \
        --region $REGION  \
        --output text  \
        --vpc-id $VPC_ID  \
        --query 'CarrierGateway.CarrierGatewayId')  \
        && echo '\nCAGW_ID='$CAGW_ID
