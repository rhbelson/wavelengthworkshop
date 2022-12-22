+++
title = "Module 4 - Configure Bastion Host"
weight = 50
+++

In the previous module, we described the Session Manager access pattern for EC2 instances in a Wavelength Zone. In this module, we will demo an alternative technique using bastion hosts.

A bastion host is a server whose purpose is to provide access to a private network from an external network, such as the Internet. In this module, we will create a bastion host in a new public subnet within the AWS Region, configure the routing for this public subnet, and deploy the EC2 instance configured with a security group allowing inbound traffic from SSH. Lastly, we will modify the Wavelength Zone EC2 instance's Security Group to allow SSH traffic from the security group of the bastion host.

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


#### Deploy the bastion / web instance

You must deploy a bastion server in order to SSH into your application instances. Remember that the carrier gateway in a Wavelength Zone only allows ingress from the carrier's 5G network. This means that in order to SSH into the API and inference servers you need to first SSH into the bastion host, and then from there SSH into your Wavelength instances.

You are also going to install the client front end application onto the bastion host.
```
     export BASTION_INSTANCE_ID=$(aws ec2 run-instances \
     --region $REGION  \
     --output text \
     --instance-type t3.medium \
     --associate-public-ip-address \
     --subnet-id $BASTION_SUBNET_ID \
     --image-id $BASTION_IMAGE_ID \
     --query 'Instances[0].InstanceId' \
     --security-group-ids $BASTION_SG_ID \
     --key-name $KEY_NAME) \
     && echo '\nBastion Instance ID' $BASTION_INSTANCE_ID
```

Take note of the bastion server public IP address you will need this later
```
      aws ec2 describe-instances --region $REGION --instance-ids $BASTION_INSTANCE_ID \
      --query 'Reservations[0].Instances[0].{"Bastion server public IP": PublicIpAddress}' 
```