+++
title = "Step 9 - Test the Application"
weight = 80
+++

In this final step you'll use Verizon's NOVA platform to access the running application via remote 5G-enabled smart phone. 

* Open a web browser on your local machine and navigate to [the Nova website](https://vzwdt.com/Nova/#/login)

* Login to the Nova platform. The email will be the AWS credit code shown on the first screen of the lab guide followed by `@vzwdt.com`. For instance in the case of the example below the email would be `PCUX6Y302YOK2A@vzwdt.com`. The password is `Welcome-123`.

![](../../images/credit_code.png)

* Hover over the 5G graphic on the left and click ***Get Started***

* You will see a grid with a list of devices click the ***Assign*** button next to one of the devices that is in the city closest to Region where you're deploying the workshop resources (Waltham for us-east-1 or San Francisco for us-west-1)

* Choose today's date and click ***Reserve***

* Your device will show up on the left hand side of the screen

![](../../images/nova_device.png)

* Click the ***Adhoc*** icon at the top left of the screen

A remote screen session will be started with the device. You can interact with the device using your mouse and keyboard, but be aware there is a bit of lag, so exercise a degree of patience. 

* Click the Chrome icon on the mobile device

* In the address bar enter the ***public IP address*** of your web server instance

* In the text box enter the carrier (public) IP address of the API server that you noted earlier

* Click ***Choose file*** 

* Click ***Files***

* Near the top under ***BROWSE FILES IN OTHER APPS*** hold your mouse button down and scroll right and click on ***Gallery***

* Choose one of the files on the device

* Click ***Process image** and the image will be re-rendered with the object detected and labeled


 
