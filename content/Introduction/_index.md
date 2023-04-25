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
Critical attributes of applications that could benefit from low-latency edge computing on AWS Wavelength include:
- **Security & Data Sovereignty**: Application requires data in close proximity to source of data production or requires traffic flow that does not traverse the public internet
- **Network Performance**: Application not only requires low latency and high throughput connection, but also prefers heightened quality-of-service or exposure of/to select network attributes
- **Measureable Impact of Latency on Customer Experience**: Application can quantify benefit of lower latency (in milliseconds), relative to the next best alternative, coupled with the perceived customer impact

You might also want to consider the following:
- **Single Carrier & B2B Applications**: As AWS Wavelength Zones today are only accessible by a single carrier at a time (see [Full List of Locations Here](aws.amazon.com/wavelength/locations/)), it is simplest to utilize AWS Wavelength for mobile B2B applications. Said differently, if you can ensure that all connected devices (e.g., IP cameras, IoT devices, mobile phones) are connected to a single carrier (currently offering AWS Wavelength), all application traffic can be served from that Wavelength Zone.
- **Integrate AWS Wavelength with Parent Region**: There may be certain services that are not natively supported within AWS Wavelength but whose underlying architectural pattern is best served from the parent region. As an example, you can utilize ML training in the region using Amazon SageMaker and run your [low-latency edge ML models](https://aws.amazon.com/blogs/machine-learning/deploy-pre-trained-models-on-aws-wavelength-with-5g-edge-using-amazon-sagemaker-jumpstart/) on AWS Wavelength.

### AWS Wavelength Customers & Partners
Check out the following customers across Media & Entertainment, Sports, IoT and beyond!

{{< youtube DJTvkEYJOTs >}}
<br>
{{< youtube uxflVEmmFHA >}}
<br>
{{< youtube 6jTIWstXMf8 >}}
<br>
{{< youtube S35QNs1Uzx0 >}}
