+++
title = "Step 7 - Configure the Inference Server Software"
weight = 60
+++

In this section you deploy Torchserve server with the fasterrcnn model. Torchserve receives the image from the API server, runs the inference, and returns the labels and bounding boxes for the items found in the image.

* You should still be SSH'd into the bastion host from the previous section, if you are not SSH back into the bastion instance. ***Note*** be sure you're starting the SSH session from the subirectory that holds your *.pem* file.


* From the bastion host SSH into your inference server using the ***private ip*** address. The user name is ***ubuntu***

    ***Note***: When you SSH into the inference server you do not need to use the -i or -A parameters.

        ssh ubuntu@<inference server private ip>

    For example:

        ssh ubuntu@10.0.0.253
        
   ***Note***: If you're having trouble with SSH key forwarding, from your local IDE, run the following first (before SSH command to bastion):
   ```
   eval "$(ssh-agent)"
   ssh-add <your-key-name>.pem
   ```

*  Update the packages on the server and install the necessary prerequisite packages.
    
        sudo apt-get update -y \
        && sudo apt-get install -y virtualenv openjdk-11-jdk gcc python3-dev python3.8-venv

 ***Note***: If you get an error that the system can't set a lock file, that means it's still installing packages after booting up the first time. You can check to see if any `apt` processes are running by entering:

        ps -ef | grep apt

If the system returns just the line below, then you should try the running the update and install commands again. If it shows more than one line, wait a few minutes and try again. 

        ubuntu   20139  3569  0 16:42 pts/1    00:00:00 grep --color=auto apt

* Create a virtual environment.
    
        mkdir inference && cd inference
        
        python3 -m venv inference
        
        source inference/bin/activate

*  Install Torchserve and its related components.
    ```
        pip3 install wheel pyyaml torch==1.12.0 torchtext==0.13.0 torchvision==0.13 sentencepiece psutil torchserve==0.6 torch-model-archiver captum nvgpu 
    ```
*  Install the inference model that the application will use.
    
        mkdir torchserve-examples && cd torchserve-examples
        
        git clone https://github.com/pytorch/serve.git
        
        mkdir model_store
        
        wget https://download.pytorch.org/models/fasterrcnn_resnet50_fpn_coco-258fb6c6.pth
        
        torch-model-archiver --model-name fasterrcnn --version 1.0 \
        --model-file serve/examples/object_detector/fast-rcnn/model.py \
        --serialized-file fasterrcnn_resnet50_fpn_coco-258fb6c6.pth \
        --handler object_detector \
        --extra-files serve/examples/object_detector/index_to_name.json
        
        mv fasterrcnn.mar model_store/

*  Export an environment variable for the private IP of your inference server. ***Be sure to substitute in the value of your inference server below***

        export INF_PRIVATE_IP=<inference server private IP>

* Create the configuration file (`config.properites`) and display it to verify. 

        echo inference_address=http://$INF_PRIVATE_IP:8080 > config.properties
        echo management_address=http://$INF_PRIVATE_IP:8081 >> config.properties
        cat config.properties


*  Start the Torchserve server
    
        torchserve --start --model-store model_store --models \
        fasterrcnn=fasterrcnn.mar --ts-config config.properties
    
    It will take a few seconds for the server to startup, when it's
    ready you should see a line that ends with:

        State change WORKER_STARTED -> WORKER_MODEL_LOADED

Leave this SSH session running so you will be able to watch the
inference server's logs to see when it receives requests from the API
server.