+++
title = "Step 8 - Deploy the API Server Software"
weight = 70
+++

In this section, you'll test to ensure your API server can communciate over the network with your inference server then you will deploy the Flask-based API server. 

##### Test communication with the inference server

1. Start a new SSH into bastion host and then SSH into the ***private IP*** of the API server instance. Remember to use the `-A` option. 

1. Set an environment variable for the private IP address of your inference server. ***Be sure to substitute the private IP of your inference server below***

        export INF_SERVER_PRIVATE_IP=<your inference server private IP>

2.  Test your inference server (being sure to substitute the INTERNAL IP of the inference instance in the second line below):
    
        curl -O https://s3.amazonaws.com/model-server/inputs/kitten.jpg
    
        curl -X POST \
        http://$INF_SERVER_PRIVATE_IP:8080/predictions/fasterrcnn \
        -T kitten.jpg
    
    You should see something similar to
    
        [
                {
                "cat": [
                        228.78250122070312,
                        82.63462829589844,
                        583.7754516601562,
                        677.3057861328125
                ],
                "score": 0.9918511509895325
                },
                {
                "car": [
                        124.42741394042969,
                        69.34326934814453,
                        270.1545715332031,
                        205.53457641601562
                ],
                "score": 0.7217231392860413
                }
        ]
       
    The inference server returns the labels of the objects it detected, and the corner coordinates of boxes that surround those objects.

Now that you have verified the API server can connect to the inference server, you can configure the API server.

##### Configure the API server

3)  Run the following command to update system package information and  install necessary prerequisites.
    
        sudo apt-get update -y \
        && sudo apt-get install -y \
        libsm6 libxrender1 libfontconfig1 virtualenv libgl1-mesa-glx

4)  Clone the Python code into the application directory
    
        mkdir apiserver && cd apiserver
    
        git clone https://github.com/mikegcoleman/flask_wavelength_api .

5)  Create and activate a virtual environment.
    
        virtualenv --python=python3 apiserver

        source apiserver/bin/activate

6)  Install necessary Python packages.
    
        pip3 install opencv-python flask pillow requests flask-cors

7)  Create a configuration file (`config_values.txt`).

        cat << EOF > config_values.txt
        http://$INF_SERVER_PRIVATE_IP:8080/predictions/fasterrcnn
        EOF     

8)  Start the application.
    
        python api.py
    
    You should see output similar to the following:
    
        * Serving Flask app "api" (lazy loading)
        * Environment: production\
        WARNING: This is a development server. Do not use it in a production
        deployment.
        Use a production WSGI server instead\
        * Debug mode: on
        * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)\
        * Restarting with stat
        * Debugger is active!
        * Debugger PIN: 311-750-351

The API server is now running. Leave this SSH server open so you can view the logs as you test the client application in the next step.