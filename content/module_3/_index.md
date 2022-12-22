+++
title = "Module 3 - Create Session Manager IAM Role"
weight = 40
+++


One of the most frequent FAQs we get from developers is how to access Wavelength Zone instances without access to the carrier network. 

Over the next 2 modules, we will describe two AWS Wavelength resource access patterns, starting with Session Manager. 

With Session Manager, you can manage your Amazon Elastic Compute Cloud (Amazon EC2) instances using an interactive one-click browser-based shell or the AWS Command Line Interface (AWS CLI).

By default, AWS Systems Manager doesn't have permission to perform actions on your instances. You must grant access by using AWS Identity and Access Management (IAM). 

For our EC2 instance, we will use an instance profile, which passes an IAM role to an Amazon EC2 instance. You can attach an IAM instance profile to an Amazon EC2 instance as you launch it or to a previously launched instance. 

In this module, we will start by creating a custom IAM policy document and trust policy.
```
    cat <<EOF > ssm-policy.json 
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "ssm:UpdateInstanceInformation",
                    "ssmmessages:CreateControlChannel",
                    "ssmmessages:CreateDataChannel",
                    "ssmmessages:OpenControlChannel",
                    "ssmmessages:OpenDataChannel"
                ],
                "Resource": "*"
            },
            {
                "Effect": "Allow",
                "Action": [
                    "s3:GetEncryptionConfiguration"
                ],
                "Resource": "*"
            },
            {
                "Effect": "Allow",
                "Action": [
                    "kms:Decrypt"
                ],
                "Resource": "$EC2_KEY"
            }
        ]
    }
    EOF
    
    cat <<EOF > ec2-trust-policy.json 
    {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "ec2.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
    }
    EOF
    SSM_POLICY_ARN=$(aws iam create-policy --policy-name ssm-wavelength-policy --policy-document file://ssm-policy.json --query Policy.Arn) && echo `SSM_POLICY_ARN=`$SSM_POLICY_ARN
```

Next, we will create the IAM role and attach the policy to it.
```
    ROLE_NAME=$(aws iam create-role --role-name wavelength-ec2-ssm --assume-role-policy-document file://ec2-trust-policy.json --query Role.RoleName) && echo `ROLE_NAME=`$ROLE_NAME
    aws iam attach-role-policy --role-name $ROLE_NAME --policy-arn $SSM_POLICY_ARN
```

Next, call the `create-instance-profile` command followed by the `add-role-to-instance-profile` command to create an IAM instance profile named Wavelength-EC2-Instance-Profile. The instance profile allows Amazon EC2 to pass the IAM role named `wavelength-ec2-ssm` to an Amazon EC2 instance when the instance is first launched:
```
    aws iam create-instance-profile --instance-profile-name Wavelength-EC2-Instance-Profile
    aws iam add-role-to-instance-profile --instance-profile-name Wavelength-EC2-Instance-Profile --role-name $ROLE_NAME
```

Lastly, attach the instance profile to the EC2 instance:
``` 
    aws ec2 associate-iam-instance-profile --iam-instance-profile Name=Wavelength-EC2-Instance-Profile --instance-id $EC2_WLZ_ID
```

To use the AWS CLI to run session commands, the Session Manager plugin must also be installed on your local machine. For information, see [(Optional) Install the Session Manager plugin for the AWS CLI](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html).

To start a session using the AWS CLI, run the following command replacing instance-id with your own information.

```
aws ssm start-session \
    --target instance-id
```