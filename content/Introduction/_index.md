+++
title = "Introduction"
weight = 10
+++

This workshop is intended to serve as a use case-agnostic overview of all of AWS Wavelength's native features and popular reference patterns, including:

-   AWS Wavelength Subnets, Route Tables Carrier Gateway
-   Amazon EC2 Instances and Carrier IP Addresses
-   Amazon EKS Self-managed Node Groups
-   Application Load Balancer (ALB)

Moreover, this workshop will cover other infrastructure-agnostic techniques, optimized for edge, including:
- Observability and Performance Testing
- Workload Placement using Pod Topology Spread Constraints
- Edge Discovery for north-south Traffic Routing

### More About AWS Wavelength
Today, to reach cloud computing environments (including the AWS Region), traffic must traverse from a base station, through the backhaul network, anchor to a mobile IP address in the core, and then egress the cellular access network to the internet at-large. As a result of each of these logical “hops,” mobile traffic incurs a latency penalty (often in milliseconds) across each hop in a mobile packets journey, amounting to ~80-100ms in round-trip delay.

To optimize latency in mobile environments today, developers can optimize the cloud application performance but what if they could optimize the network itself?

Instead of needing a “5G expert” on your development teams, AWS Wavelength abstracts away the complexity of 5G networks so developers can focus on building applications. In doing so, AWS compute and storage services are brought within the mobile core to reduce latency, increase the security posture of mobile applications, and minimize the “hops” between source (i.e., mobile device) and destination (i.e., edge cloud).

{{< figure src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2022/12/15/Screen-Shot-2022-12-15-at-12.11.20-PM.png" title="AWS Wavelength Reference Architecture">}}