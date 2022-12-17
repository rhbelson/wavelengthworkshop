+++
title = "Module 5 - Create EKS Cluster"
weight = 60
+++

With the VPC and underlying networking and security deployed you can nowmove on to deploying your API, inference, and bastion instances.   You will install and configure the software components in subsequent steps. 

##### Deploy the API instance. 

The API server will be a t3.instance based on a standard Ubuntu 18.04 AMI.

*  Deploy the API instance

        export API_INSTANCE_ID=$(aws ec2 run-instances \
        --region $REGION \
        --output text \
        --instance-type t3.medium \
        --network-interface '[{"DeviceIndex":0,"NetworkInterfaceId":"'$API_ENI_ID'"}]' \
        --image-id $API_IMAGE_ID \
        --query 'Instances[0].InstanceId' \
        --key-name $KEY_NAME) \
        && echo '\nAPI Server Instance ID '$API_INSTANCE_ID

* Take note of the API server private IP

        aws ec2 describe-instances --region $REGION --instance-ids $API_INSTANCE_ID \
        --query 'Reservations[0].Instances[0].{"API server private IP": PrivateIpAddress}'  


##### Deploy the inference intance

The inference server is a g4dn.2xlarge running the AWS deep learning AMI.

* Deploy the inference instance

        export INF_INSTANCE_ID=$(aws ec2 run-instances \
         --region $REGION \
         --instance-type g4dn.2xlarge \
         --output text \
         --network-interface '[{"DeviceIndex":0,"NetworkInterfaceId":"'$INFERENCE_ENI_ID'"}]' \
         --image-id $INFERENCE_IMAGE_ID \
         --query 'Instances[0].InstanceId' \
         --block-device-mappings '[{"DeviceName":"/dev/sda1","Ebs":{"VolumeType":"gp2"}}]' \
         --key-name $KEY_NAME) \
         && echo '\nInference Server Instance ID' $INF_INSTANCE_ID

* Take note of the Inference Servers Private IP

        aws ec2 describe-instances --region $REGION --instance-ids $INF_INSTANCE_ID \
        --query 'Reservations[0].Instances[0].{"Inference server private IP": PrivateIpAddress}'      

##### Deploy the bastion / web instance

You must deploy a bastion server in order to SSH into your application
instances. Remember that the carrier gateway in a Wavelength Zone only
allows ingress from the carrier's 5G network. This means that in order
to SSH into the API and inference servers you need to first SSH into the
bastion host, and then from there SSH into your Wavelength instances. 

You are also going to install the client front end application onto the
bastion host. 

*  Create the bastion / web instance

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

* Take note of the bastion server public IP address you will need this later

        aws ec2 describe-instances --region $REGION --instance-ids $BASTION_INSTANCE_ID \
        --query 'Reservations[0].Instances[0].{"Bastion server public IP": PublicIpAddress}' 
