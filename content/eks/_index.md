+++
title = "EKS on AWS Wavelength"
weight = 30
+++

In this section, learn how to extend existing Amazon EKS Clusters to AWS Wavelength Zones with self-managed worker nodes.

### Reference Architecture
Today, Amazon EKS on AWS Wavelength doesn’t support managed node groups or AWS Fargate. As such, it’s critical to note the following design considerations for your cluster:

- **Amazon EKS Control Plane:** Much like traditional clusters, Amazon EKS requires two VPC subnets to launch the Control Plane. However, neither of the two subnets can include an AWS Wavelength Zone; and unlike AWS Outposts, Local clusters for Amazon EKS is not supported.

- **EKS Worker Nodes:** To create node groups, we need to launch AWS Auto Scaling Groups running an Amazon EKS-optimized Amazon Machine Image (AMI). This AMI has the kubelet configured to connect back to the Amazon EKS Control Plane in the Parent Region.

- **Service Link:** To connect the worker nodes back to the Control Plane across each AWS Wavelength Zone, the Service Link connects each AWS Wavelength Zone back the Parent Region. Unlike AWS Direct Connect, this connectivity link is abstracted away from the customer. Traffic over the service link is charged similarly to inter-AZ traffic within a region.
{{< figure src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2022/12/15/Screen-Shot-2022-12-15-at-12.11.34-PM.png" title=">EKS on AWS Wavelength" >}}


### More Resources
- [AWS Containers Blog, EKS on AWS Wavelength)](https://aws.amazon.com/blogs/containers/deploy-geo-distributed-amazon-eks-clusters-on-aws-wavelength/)
- [Containers on AWS Wavelength, Pluralsight](https://www.pluralsight.com/courses/containers-aws-wavelength): In this course (32min) by Nigel Poulton, learn how to deploy edge applications to Amazon EKS clusters with nodes in AWS Wavelength zones.
