---
title: "AWS Wavelength Getting Started Guide"
chapter: true
weight: 1
---

# AWS Wavelength Getting Started Guide

* In this user guide, learn the fundamentals of 5G mobile networks, mobile edge computing (MEC) alongside a step-by-step guide to deploying your first application AWS Wavelength. Moreover, learn how to measure application performance and troubleshoot popular FAQs and configurations.

### What is AWS Wavelength?
<div>
<iframe width="560" height="315" src="https://www.youtube.com/embed/EhMqwPqPzcY" title="Introduction to AWS Wavelength" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
</div>

- AWS Wavelength is an infrastructure service, powered by Amazon Web Services (AWS), that enables developers to build applications within the 5G mobile network to deliver lower latency, more immersive experiences. 
- AWS Wavelength offers developers standard AWS compute and storage services at the edge of Telecommunications providers' 5G networks via the extension a virtual private cloud (VPC) to one or more Wavelength Zones. 
- An AWS Wavelength Zone is a zone in a carrier 5G network where the AWS Wavelength infrastructure is deployed. Each Wavelength Zone is associated with a parent AWS Region. 
- As an example, the Bell Toronto Wavelength Zone (ca-central-1-wl1-yto-wlz-1) is associated with Canada (Central) AWS Region.
- While AWS Wavelength is optimized for 5G networks, any 4G or 5G-connected device (for the given carrier) can access the AWS Wavelength Zone

* To learn more about configuring AWS Wavelength applications, you can always visit the [developer guide](https://docs.aws.amazon.com/pdfs/wavelength/latest/developerguide/aws-wavelength-developer-guide.pdf). 

### More Resources
- [Introduction to AWS Wavelength, Pluralsight](https://www.pluralsight.com/courses/aws-wavelength-introduction): In this course (26min) by Andru Estes, learn the fundamentals of AWS Wavelength and deploy a simple Amazon EC2 Flask application in an AWS Wavelength Zone
- [Containers on AWS Wavelength, Pluralsight](https://www.pluralsight.com/courses/containers-aws-wavelength): In this course (32min) by Nigel Poulton, learn how to deploy edge applications to Amazon EKS clusters with nodes in AWS Wavelength zones.
- [Verizon 5G Edge Tutorials, GitHub](https://github.com/Verizon/5GEdgeTutorials/): In this code repository, Verizon created a number of starter projects, tutorials, and infrastructure templates that get your AWS Wavelength infrastructure up-and-running in seconds.