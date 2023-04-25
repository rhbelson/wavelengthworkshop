+++
title = "Introduction"
weight = 10
+++

This workshop is intended to serve as a use case-agnostic overview of all of AWS Wavelength's native features and popular reference patterns, including:

-   **Networking**: AWS Wavelength Subnets, Carrier Route Tables, Carrier Gateway and Carrier IP Addresses
-   **Compute**: Amazon EC2, Amazon EKS, and Application Load Balancer (ALB)

Moreover, this workshop will cover other infrastructure-agnostic techniques, optimized for edge, including observability, workload placement and edge discovery.

### More About AWS Wavelength
Today, to reach cloud computing environments (including the AWS Region), traffic must traverse from a base station, through the backhaul network, anchor to a mobile IP address in the core, and then egress the cellular access network to the internet at-large. As a result of each of these logical “hops,” mobile traffic incurs a latency penalty (often in milliseconds) across each hop in a mobile packets journey, amounting to ~80-100ms in round-trip delay.

![AWS Wavelength and Parent Region](./introduction/wl_ref.png)

To optimize latency in mobile environments today, developers can optimize the cloud application performance but what if they could optimize the network itself?

Instead of needing a “5G expert” on your development teams, AWS Wavelength abstracts away the complexity of 5G networks so developers can focus on building applications. In doing so, AWS compute and storage services are brought within the mobile core to reduce latency, increase the security posture of mobile applications, and minimize the “hops” between source (i.e., mobile device) and destination (i.e., edge cloud).

- AWS Wavelength is an infrastructure service, powered by Amazon Web Services (AWS), that enables developers to build applications within the 5G mobile network to deliver lower latency, more immersive experiences. AWS Wavelength offers developers standard AWS compute and storage services at the edge of Telecommunications providers' 5G networks via the extension a virtual private cloud (VPC) to one or more Wavelength Zones. 
- An AWS Wavelength Zone is a zone in a carrier 5G network where the AWS Wavelength infrastructure is deployed. Each Wavelength Zone is associated with a parent AWS Region. While AWS Wavelength is optimized for 5G networks, any 4G or 5G-connected device (for the given carrier) can access the AWS Wavelength Zone

### When to Use AWS Wavelength
Critical attributes of applications that could benefit from low-latency edge computing on AWS Wavelegnth include:
- **Security & Data Sovereignty**: Application requires data in close proximity to source of data production or requires traffic flow that does not traverse the public internet
- **Network Performance**: Application not only requires low latency and high throughput connection, but also prefers heightened quality-of-service or exposure of/to select network attributes
- **Measureable Impact of Latency on Customer Experience**: Application can quantify benefit of lower latency (in milliseconds), relative to the next best alternative, coupled with the perceived customer impact

### Customers on AWS Wavelength Today
Check out the following customers across Media & Entertainment, Sports, IoT and beyond!
{{< youtube DJTvkEYJOTs title="Immersive Media Experience for Golf Carts" >}}
<br>
{{< youtube uxflVEmmFHA title="Crowd Analytics Use Case for Smart Venues" >}}
<br>
{{< youtube 6jTIWstXMf8 title="Sports Analytics Use Case for Stadiums" >}}
<br>
{{< youtube S35QNs1Uzx0 title="Additional Sports Analytics Use Case for Stadiums" >}}
