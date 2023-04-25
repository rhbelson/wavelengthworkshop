+++
title = "Workload Placement and Edge Discovery"
weight = 40
+++

In this section, learn about the critical Day 2 Operations associated with AWS Wavelength across workload placement and edge discovery.

### Background 
AWS Wavelength operates in a hub-and-spoke architecture where all spokes can only communicate within their zone itself or back to the hub (i.e., Parent Region). This introduces challenges for developers looking to take advantage of low-latency geo-distributed applications at scale.

**Workload placement:** Applications with low latency requirements must ensure that all latency-sensitive components are scheduled to each edge zone, as desired by the customer. To illustrate the impact of workload placement in a geo-distributed setting, consider the architecture of an automotive company wishing to reduce the latency for a machine learning-based workflow for automatic pedestrian detection or vehicle-to-anything (V2X) connectivity. Perhaps the two-tier web app consists of an Application Programming Interface (API) proxy and machine learning inference service, each geo distributed across every available Availability Zone (AZ) and Local Zone (LZ) to create uniform, low latency access for vehicles within a given AWS Region. In this example, it would be critical to have a copy of the inference endpoint in each topology (i.e., Wavelength Zone), but not necessarily each node (if there are multiple nodes per Wavelength Zone subnet). In this module, we will demonstrate an example of workload placement using EKS.

**Edge discovery:** Once the application is deployed, developers often ask for the optimal selection criteria, heuristic, or algorithms – often encompassing network latency, available network bandwidth, or network topology – to determine the optimal application endpoint for a given client session. In this module, we will demonstrate how to use Carrier Edge Discovery APIs alongside your AWS environment to discover the optimal edge zone.

### More Resources
- [Architecting Geo-Distributed Mobile Edge Application With Consul)](https://www.youtube.com/watch?v=-vRNwZ0Uofc)
