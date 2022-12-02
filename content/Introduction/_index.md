+++
title = "Introduction"
weight = 10
+++

The following tutorial guides you through deploying an object detection
application that is comprised of three components:

-   A Wavelength-hosted API endpoint (using Flask)
-   A Wavelength-hosted Inference server (running Torchserve)
-   React-based client web app running

![](../../images/final_diagram.png)

The API server is built using Python and Flask, and runs on a t3.medium
instance based upon a standard Ubuntu 18.04 image. It accepts an image
from the client application running on a device connected to the
carriers 5G mobile network, which it then forwards to the inference
server. The inference server returns the detected object along with
coordinates for that object (or an error if it can't detect any
objects). The API server adds a text label and bounding boxes to the
image and returns it to the mobile client.

The inference server runs
[Torchserve](https://github.com/pytorch/serve), an open source project
that provides a flexible and easy way to serve up PyTorch models. Object
detection is done using a Faster R-CNN model. You then deploy it on a
g4dn.2xlarge instance running the AWS deep learning Amazon Machine Image
(AMI).

You will use  a React web application to access the application services. 

Wavelength is designed to provide access to services and applications
that require low latency. It's important to note that you don't need to
deploy your entire application in a Wavelength Zone. You only need to
deploy parts of your application that benefit from being deployed in the
Wavelength Zone -- i.e., application components requiring low latency.

In the case of the demo application, the API and inference servers are
located in the Wavelength Zone because one of the design goals of the
application is low-latency processing of the inference requests.

On the other hand, because the web application is only serving a single
small static web page, it does not have the same latency requirements as
the inference processing. For that reason, it's hosted in the Region
instead of the Wavelength Zone.

