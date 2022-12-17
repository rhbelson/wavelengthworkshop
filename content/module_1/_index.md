+++
title = "Module 1 - Extend VPC to Multiple Wavelength Zones"
weight = 20
+++


In this module, we will demonstrate how to extend your Amazon VPC to multiple Wavelength Zones while continuing to use `us-west-2` as our AWS Region of choice.

To start, define the environment variables that will be used to create our Wavelength Zone subnets. In this example, we would like to include the San Francisco, Los Angeles and Las Vegas Wavelength Zones as part of our application so we'll export 3 environment variables corresponding to Availability Zone ID of that subnet.

    ```
    export REGION="us-west-2"
    export WAVELENGTH_ZONE_1="us-west-2-wl1-sfo-wlz-1"
    export WAVELENGTH_ZONE_2="us-west-2-wl1-lax-wlz-1"
    export WAVELENGTH_ZONE_3="us-west-2-wl1-las-wlz-1"
    ```

As an example, `WAVELENGTH_ZONE_1` corresponds to the San Francisco Wavelength Zone. The complete mapping of Availability Zone IDs to AWS Wavelength cities can be found in the [AWS Wavelength Locations](https://aws.amazon.com/wavelength/locations/) page. 
        

Next, we will use the AWS CLI to create the VPC. Note that there is nothing Wavelength-specific in this step; more information about creating VPCs can be found in the [Amazon VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html).
    ```
    export VPC_ID=$(aws ec2 create-vpc  \
    --region $REGION  \
    --output text  \
    --cidr-block 10.0.0.0/16  \
    --tag-specifications ResourceType=vpc,Tags='[{Key=Name,Value="Wavelength-Workshop-VPC"}]' \
    --query 'Vpc.VpcId')  \
    && echo 'VPC_ID='$VPC_ID
    ```
    
With the command above, we create an Amazon VPC with the `10.0.0.0/16` CIDR range, name the VPC `Wavelength-Workshop-VPC` and print out the VPC ID at the very end.

```
Sample Output:

Admin:~/environment $ export VPC_ID=$(aws ec2 create-vpc  \
>     --region $REGION  \
>     --output text  \
>     --cidr-block 10.0.0.0/16  \
>     --tag-specifications ResourceType=vpc,Tags='[{Key=Name,Value="Wavelength-Workshop-VPC"}]' \
>     --query 'Vpc.VpcId')  \
>     && echo 'VPC_ID='$VPC_ID
VPC_ID=vpc-01743658ac51bc14b

```

Next, we will create a Carrier Gateway. A carrier gateway serves two purposes:
- It allows inbound traffic from a carrier network in a specific location
- It allows outbound traffic to the carrier network and internet. There is no inbound connection configuration from the internet to a Wavelength Zone through the carrier gateway
    ```
    export CAGW_ID=$(aws ec2 create-carrier-gateway  \
    --region $REGION  \
    --output text  \
    --vpc-id $VPC_ID  \
    --query 'CarrierGateway.CarrierGatewayId')  \
    && echo 'CAGW_ID='$CAGW_ID
    ```

With the command above, you will create and automatically attach a Carrier Gateway to the VPC you created in the prior step.

```
Sample Output:

export CAGW_ID=$(aws ec2 create-carrier-gateway  \
    --region $REGION  \
    --output text  \
    --vpc-id $VPC_ID  \
    --query 'CarrierGateway.CarrierGatewayId')  \
    && echo 'CAGW_ID='$CAGW_ID
```