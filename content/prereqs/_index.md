+++
title = "Opt-in to AWS Wavelength"
weight = 15
+++

Prerequisites
=============

To complete the walkthrough below you will need:

-   The [AWS CLI version 2](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) installed on your local machine. Ensure it's the latest
    version so it supports Wavelength.

-   An AWS account with sufficient privileges to create VPC
    resources (instances, subnets, etc).

To start, let's opt-in to all of the AWS Wavelength Zones. To do so, first select the desired AWS Region, which can be found by visiting the available [AWS Wavelength Locations](https://aws.amazon.com/wavelength/locations/).
```
export REGION=<your-region>
```

In our example, we will use the `us-east-1` corresponding to the AWS Wavelength Zones connected to the Verizon 5G Edge network.

To allow-list access, we will use the `modify-availability-zone-group` method in the EC2 API to automatically opt-in to all AWS Wavelength Zones at once.

```
export REGION=us-east-1
aws ec2 modify-availability-zone-group --group-name ${REGION}-wl1 --opt-in-status opted-in --region $REGION
```

