+++
title = "Step 6 - Configure the Web host"
weight = 60
+++

The last server you deployed serves two purposes. It acts as the bastion host allowing you to SSH into your other two servers, and it serves the client web app. In this section you'll install that web app.

* Open a new terminal window. This will help preserve access to the environment variables you created earlier, which can be useful in trouble shooting. 

* On your local machine change into the directory that holds the  *.pem* for they key pair you specified when you created the instances. 

* SSH into bastion host (the user name is ***ec2-user***) from your new terminal window. 

    ***Note:*** In order to be able to easily SSH from the bastion host to the inference or API servers you will want to use the -A (agent forwarding) parameter when starting your SSH session e.g.:

        ssh -i key.pem -A ec2-user@<bastion public ip address>

    For example:

        ssh -i my_key.pem -A ec2-user@192.168.0.1

*  Download dependencies for git and npm
        sudo yum install -y git
        curl -sL https://rpm.nodesource.com/setup_14.x | sudo bash -
        sudo yum install nodejs

*  Clone the GitHub repo with the React code
    
        git clone https://github.com/mikegcoleman/react-wavelength-inference-demo.git

*  Install the dependencies

        cd react-wavelength-inference-demo && npm install

* Build the web page

        npm run build

*  Copy the page into web servers root directory and configure nginx web server
        sudo yum install -y httpd
        sudo systemctl start httpd
        sudo systemctl enable httpd

        sudo mkdir /var/www/html
        cp -r ./build/* /var/www/html 

*  Test that the web app is running correctly by navigating to the
    public IP address of your bastion instance
