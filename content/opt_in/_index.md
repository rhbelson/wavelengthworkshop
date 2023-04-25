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

In our example, we will use the `us-west-2` corresponding to the AWS Wavelength Zones connected to the Verizon 5G Edge network.

To allow-list access, we will use the `modify-availability-zone-group` method in the EC2 API to automatically opt-in to all AWS Wavelength Zones at once.

```
export REGION=us-west-2
aws ec2 modify-availability-zone-group --group-name ${REGION}-wl1 --opt-in-status opted-in --region $REGION
```

To view all the Availability Zones and Wavelength Zones, now availability for use within your account, run the following:
```
aws ec2  describe-availability-zones --region $REGION
```

In our example within `us-west-2`, we will see the following as the output: 
```
[
    "us-west-2a", 
    "us-west-2b", 
    "us-west-2c", 
    "us-west-2d", 
    "us-west-2-wl1-den-wlz-1", 
    "us-west-2-wl1-las-wlz-1", 
    "us-west-2-wl1-lax-wlz-1", 
    "us-west-2-wl1-phx-wlz-1", 
    "us-west-2-wl1-sea-wlz-1", 
    "us-west-2-wl1-sfo-wlz-1"
]
```

**Note**: There is no charge for opting-in to AWS Wavelength Zones. After opting-in, you can continue completing this hands-on workshop.