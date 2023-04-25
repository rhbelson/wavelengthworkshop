+++
title = "Module 2 - Create EC2 Instance with Carrier IP"
weight = 30
+++

In this section, we will need to configure 3 basic prerequisites for our Amazon EC2 Instance.
1) Create a Security Group to attach to our Amazon EC2 instance.
2) Create a Key Pair so that we can remotely access our instance via Secure Shell Protocol (SSH).
3) Retrieve an Amazon Machine Image (ID) with which to launch our EC2 instance

First, start by creating the application security group.

```
        export WAVELENGTH_SG_ID=$(aws ec2 create-security-group \
        --region $REGION \
        --output text \
        --group-name wavelength-app-sg \
        --description "Security group for Wavelength Zone application" \
        --vpc-id $VPC_ID \
        --query 'GroupId') \
        && echo 'WAVELENGTH_SG_ID='WAVELENGTH_SG_ID
```
Next, create an ingress rule to allow SSH from your current IP address. If you want to access SSH from anywhere change `$(curl https://checkip.amazonaws.com)/32` to `0.0.0.0/0` - but be aware that this allows access from any IP address and is considered a security anti-pattern. 

```
        aws ec2 authorize-security-group-ingress \
        --region $REGION \
        --group-id $WAVELENGTH_SG_ID \
        --protocol tcp \
        --port 22 \
        --cidr $(curl https://checkip.amazonaws.com)/32
```
Next, create a Key Pair and save it to your local filesystem. Note in this example, we will store the key material in a file called `edge-key.pem`. By default, this private key will be over-permissive, so we will also change the permissions for the private key to read-only for the user.

```
        aws ec2 create-key-pair --key-name edge-key --query 'KeyMaterial' --output text > edge-key.pem
        chmod 400 edge-key.pem
```     
To ensure that we configured the key appropriately, we can call the `describe-key-pair` method to retrieve our key pair.

```
        export EC2_KEY="edge-key"
        
        aws ec2 describe-key-pairs --key-names $EC2_KEY
        
        Sample output:
        {
            "KeyPairs": [
                {
                    "KeyPairId": "key-0408c8d7a52ba2a5f",
                    "KeyFingerprint": "94:47:74:b4:e1:a2:54:93:61:7c:d8:57:c4:28:db:c1:a7:d5:6a:82",
                    "KeyName": "edge-key",
                    "Tags": []
                }
            ]
        }
```

Next, let's use the AWS Systems Manager (SSM) API to query the latest Amazon Linux AMI ImageID value. Each Region has an exact replica namespace containing its Region-specific ImageID value.

```
        AMI_ID_LINUX=$(aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 --query 'Parameters[0].[Value]' --output text) && echo `AMI_ID_LINUX=`$AMI_ID_LINUX
```

Now, we're ready to launch our Amazon EC2 instance in the Wavelength Zone! To do so, we will need to provide the `SubnetId` we would like to use, the Security Group we created earlier, the AMI ID and the key name.
```
        EC2_WLZ_ID=$(aws ec2 run-instances --network-interfaces '[{"DeviceIndex":0, "SubnetId": "'"$SFO_WLZ_SUBNET"'", "AssociateCarrierIpAddress": true, "Groups": ["'"$WAVELENGTH_SG_ID"'"]}]' --image-id $AMI_ID_LINUX --instance-type t3.medium  --key-name $EC2_KEY --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=Wavelength-Getting-Started-EC2-Instance}]' --query 'Instances[0].InstanceId' --output text)        
```

If you navigate to the EC2 section of your AWS Console, you will now see an instance named `Wavelength-Getting-Started-EC2-Instance`. Another way to ensure that the Amazon EC2 instance was created properly is to query the Carrier IP address, like this:
```
        EC2_WLZ_IP=$(aws ec2 describe-instances --instance-id $EC2_WLZ_ID --query 'Reservations[*].Instances[*].NetworkInterfaces[*].Association.CarrierIp' --output text)
        echo $EC2_WLZ_IP
```