+++
title = "Module 9 - Edge Discovery"
weight = 80
+++

In this module, learn more about edge discovery and why it can be valuable for multiple edge-based applications.

### Edge Discovery Service Overview
Edge discovery services are APIs that allows you to register any application resources, such as databases, queues, microservices, and other cloud resources, with custom names. Each custom name is a unique identifier referred to as a serviceEndpoints object, consisting of the carrier-facing application service endpoint metadata. Each serviceEndpoint object consists of the carrier IPv4 address (IPv6 not currently supported in AWS Wavelength), an optional FQDN and port, among other metadata. 

When a mobile client seeks to identify the most optimal edge endpoint, the client must first determine its CG-NAT Public IP, through TURN server or other tools (e.g., ifconfig.me). After determining its IP address, it will will pass this value to the required UEIdentity attribute alongside the specific serviceEndpoints identifier to retrieve the most optimal service endpoint. To learn more about the API reference, visit the [5G Future Forum API specifications](https://www.5gff.org/apis/) or check out this blog below.

{{< figure src="https://d2908q01vomqb2.cloudfront.net/c5b76da3e608d34edb07244cd9b875ee86906328/2023/04/04/Architecting-multi-carrier-interoperability-with-Edge-Discovery-APIs-on-AWS-Wavelength_demo-architecture.png" title=">Mutli-carrier Edge Discovery Deep Dive" >}}

### Example Configuration
In this example, we will configure the Verizon Edge Discovery Service API for our US-based Wavelength Zone application.

**Step 1:** Sign-up for the Edge Discovery Service (EDS)<br>
For the first step, you'll need to create account on our [5G Edge Portal](https://5gedge.verizon.com/) to access a key pair.

Once logged in, visit the `API Keys` subsection of the developer portal to get started.

![5G Edge Developer Portal Login](./day2_ops/module_9/console_login.png)

**Step 2:** Create Key Pair for the Edge Discovery Service<br>

Next, click `Add new key set` in the top-right corner to generate your key pair. You'll notice the application key and secret key and automatically generated below.

![5G Edge Key Pair](./day2_ops/module_9/api_key.png)

Once you have generated your key pair, take note of your `Key` and your `Secret` pair from the console.

Let's export our application and secret keys to environment variables.
```
    export EDS_APP_KEY=<your-app-key>
    
    export EDS_SECRET_KET=<your-secret-key>
```


### Configure EDS API to Register Carrier Targets

**Step 1**: Extract AWS Wavelength Carrier IP Addresses<br>
In this next step, we will register our Carrier-facing endpoints to the edge discovery service. To start, we'll navigate to the directory of our [Python SDK for the Verizon Edge Discovery Service (EDS)](https://github.com/Verizon/5GEdgeTutorials/tree/main/edge-discovery/eds-python), developed by the Verizon 5G Edge team, to automatically register our endpoints to the service registry.

```
    git clone https://github.com/Verizon/5GEdgeTutorials.git
    cd edge-discovery/eds-python
```

Next, we'll invoke the EDS API to register the carrier targets.
```
    wlzs=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=wavelength-eks-nodegroup*" \
    --query 'Reservations[].Instances[].Placement.AvailabilityZone' \
    --output text)
    carrier_ips=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=wavelength-eks-nodegroup*" \
    --query 'Reservations[].Instances[].NetworkInterfaces[].Association.CarrierIp' \
    --output text)
    
    echo $wlzs, $carrier_ips
```
**Step 2**: Populate the Verizon Edge Discovery Service<br>
Next, visit the file `edsSetup.py` file and be sure to add your EDS key pair on lines 24-25:
```
    access_token=vzEdgeDiscovery.authenticate(
        appKey="<your-key>",
        secretKey="<your-secret-key>")
```

Then run `edsSetup.py` to populate the service registry with the Carrier IP addresses.
```
    python3 edsSetup.py --carrier_ips "$carrier_ips" --wlzs "$wlzs"
```

Be sure to take note of the output of this script, the `serviceEndpointsId` object. This will be critical for the Atlas Function to invoke on the mobile client's behalf for edge discovery.

```
    # Output from EDS
    Creating service registry with Carrier IP information...
    Your Create Service Registry response : <Response [200]>
    Your Service Endpoints ID: 87356e82-2b16-460c-8bbf-8f975f81fb47
    Your Application ID: ['endpoint_ny', 'endpoint_bo']
    
    # CLI Input
    export EDS_SERVICE_ENDPOINTS_ID=<your-service-endpoints-id>
```

**Step 3**: Identify Optimal Endpoint from Mobile Device

From a mobile device, authenticate to EDS and discover the closest carrier endpoint.

Note that the `discover_closest_edge_zone` method requires a `ue_identity` parameter, which corresponds to yoour mobile device's IP address (ex: 174.249.33.62). This can be retrieved by navigating to `ifconfig.me` or can be queried using `curl ifconfig.me` from the command line.
Additionally, please note that this API can be queried directly from the public internet, not just the carrier network.

```
    from vz_edge_discovery import VzEdgeDiscovery
    my_obj = VzEdgeDiscovery()
    access_token=my_obj.authenticate(app_key="<your-app-key>", secret_key="<your-secret-key>")
    closest_zone=my_obj.discover_closest_edge_zone(access_token=access_token, service_endpoints_id="<your-service-registry>", ue_identity="<your-mobile-ip-address>")
```

The API will return both the optimal Carrier IP address and edge zone.

```
    Selecting closest Mobile Edge Computing (MEC) endpoint...
    Your Closest Edge Zone: us-east-1-wl1-bos-wlz-1
    Your Closest IP Address: 155.146.96.195
```
