+++
title = "Step 6 - Configure the Web host"
weight = 60
+++

The last server you deployed serves two purposes. It acts as the bastion host allowing you to SSH into your other two servers, and it serves the client web app. In this section you'll install that web app.

1)  SSH into bastion host (the user name is ***bitnami***). 

    ***Note:*** In order to be able to easily SSH from the bastion host to the inference or API servers you will want to use the -A (agent forwarding) parameter when starting your SSH session e.g.:

            ssh -i /path/to/key.pem -A bitnami@<bastion public ip address>

    For example:

            ssh -i my_key.pem -A bitnami@192.168.0.1

2)  Clone the GitHub repo with the React code
    
            git clone https://github.com/mikegcoleman/react-wavelength-inference-demo.git

3)  Install the dependencies

            cd react-wavelength-inference-demo && npm install

4)  Build the web page

            npm run build

5)  Copy the page into web servers root directory

            cp -r ./build/* /home/bitnami/htdocs

6)  Test that the web app is running correctly by navigating to the
    public IP address of your bastion instance
