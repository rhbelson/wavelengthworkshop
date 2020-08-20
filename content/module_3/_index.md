+++
title = "Step 3 - Create Subnets and Routes"
weight = 40
+++


In the following steps you'll create two subnets along with their associated routing tables and routes. One subnet is associated with the Wavelength Zone, and will only be accesible over the carrier network. The other subnet will hold your bastion host, and will accessible via the Internet. 

##### Create the subnet zone for the Wavelength Zone

*  Deploy a subnet in the Wavelength Zone

        export WL_SUBNET_ID=$(aws ec2 create-subnet \
        --region $REGION \
        --output text \
        --cidr-block 10.0.0.0/24 \
        --availability-zone $WL_ZONE \
        --vpc-id $VPC_ID \
        --query 'Subnet.SubnetId') \
        && echo '\nWL_SUBNET_ID='$WL_SUBNET_ID

*  Create the route table for the Wavelength subnet

        export WL_RT_ID=$(aws ec2 create-route-table \
        --region $REGION \
        --output text \
        --vpc-id $VPC_ID \
        --query 'RouteTable.RouteTableId') \
        && echo '\nWL_RT_ID='$WL_RT_ID

*  Associate the route table with the subnet. 

        aws ec2 associate-route-table \
        --region $REGION \
        --route-table-id $WL_RT_ID \
        --subnet-id $WL_SUBNET_ID

* Add a default route to the route table that routes traffic through the carrier gateway. 

        aws ec2 create-route \
        --region $REGION \
        --route-table-id $WL_RT_ID \
        --destination-cidr-block 0.0.0.0/0 \
        --carrier-gateway-id $CAGW_ID

##### Create the bastion subnet

*  Deploy a subnet into the VPC letting AWS pick the availability zone. 

        BASTION_SUBNET_ID=$(aws ec2 create-subnet \
        --region $REGION \
        --output text \
        --cidr-block 10.0.1.0/24 \
        --vpc-id $VPC_ID \
        --query 'Subnet.SubnetId') \
        && echo '\nBASTION_SUBNET_ID='$BASTION_SUBNET_ID

*  Deploy the bastion subnet route table.

        export BASTION_RT_ID=$(aws ec2 create-route-table \
        --region $REGION \
        --output text \
        --vpc-id $VPC_ID \
        --query 'RouteTable.RouteTableId') \
        && echo '\nBASTION_RT_ID='$BASTION_RT_ID

* Associate the route table with the subnet. 

        aws ec2 associate-route-table \
        --region $REGION \
        --subnet-id $BASTION_SUBNET_ID \
        --route-table-id $BASTION_RT_ID


* Create a default route that routes traffic to the Internet via the Internet gateway.

        aws ec2 create-route \
        --region $REGION \
        --route-table-id $BASTION_RT_ID \
        --destination-cidr-block 0.0.0.0/0 \
        --gateway-id $IGW_ID



*  Modify the bastion subnet to assign public IPs by default

        aws ec2 modify-subnet-attribute \
        --region $REGION \
        --subnet-id $BASTION_SUBNET_ID \
        --map-public-ip-on-launch