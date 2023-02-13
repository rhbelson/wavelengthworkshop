+++
title = "Module 8 - Workload Placement"
weight = 70
+++

In this module, we will demonstrate how to use Pod Topology Spread Constraints, which specifies how pods should be placed within the cluster based on certain attributes of the pods. This can be useful for a variety of reasons, such as:
- Ensuring that pods are evenly distributed across different availability zones to improve reliability
- Ensuring that certain pods are only scheduled on specific nodes for performance or compliance reasons. 

To use this native feature of Kubernetes, let's inspect the current EKS worker nodes in your cluster by running `kubectl get nodes --show-labels`.
- As one of the many labels that the nodes will inherit, `topology.kubernetes.io/zone` is a label that is auto-assigned by Amazon EKS witha a value corresponding to the node's availability zone (e.g., `us-east-1-wl1-bos-wlz-1`)

Since Pod Topology Spread Constraints rely on unique label **keys**, we need to add a custom label for each node differentiating its topology, corresponding to each node's availability zone.

In our example, consider the following setup:
- The 10.0.1.52 node is in `us-east-1a`
- The 10.0.10.48 is in `us-east-1-wl1-bos-wlz-1`
- The 10.0.11.124 is in `us-east-1-wl1-nyc-wlz-1`
- The 10.0.2.160 node is in `us-east-1b`
```
NAME                          STATUS   ROLES    AGE   VERSION
ip-10-0-1-52.ec2.internal     Ready    <none>   11m   v1.21.2-eks-55daa9d 
ip-10-0-10-48.ec2.internal    Ready    <none>   10m   v1.21.2-eks-55daa9d
ip-10-0-11-124.ec2.internal   Ready    <none>   10m   v1.21.2-eks-55daa9d
ip-10-0-2-160.ec2.internal    Ready    <none>   11m   v1.21.2-eks-55daa9d
````
We want a bash script to automatically iterate through our worker nodes and label each node with a <key,value> pair corresponding to <`availability-zone`, `key`>, where **key** can be any value.
One way to do this is to create a file, `add-topologies.sh`, with the following:
```
NODES=$(kubectl get nodes -o json)

for node in $(echo "$NODES" | jq -r '.items[].metadata.name'); do
  LABEL=$(echo "$NODES" | jq -r --arg node_name "$node" '.items[] | select(.metadata.name == $node_name) | .metadata.labels."topology.kubernetes.io/zone"')
  if [[ "$LABEL" != *"wlz"* ]];
  then
    LABEL="parent-region"
  fi
  echo "Node: $node, Label: $LABEL"
  kubectl label nodes "$node" "eks.topology=$LABEL" 
done


# Sample Output:
Node: ip-10-0-1-52.ec2.internal, Label: us-east-1a
node/ip-10-0-1-52.ec2.internal labeled
Node: ip-10-0-10-48.ec2.internal, Label: us-east-1-wl1-nyc-wlz-1
node/ip-10-0-10-48.ec2.internal labeled
Node: ip-10-0-11-124.ec2.internal, Label: us-east-1-wl1-bos-wlz-1
node/ip-10-0-11-124.ec2.internal labeled
Node: ip-10-0-2-160.ec2.internal, Label: us-east-1b
node/ip-10-0-2-160.ec2.internal labeled
```
As a sample workload, let's use nginx and create a file `nginx.yaml`:

```
cat > ptsc.yaml <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 6
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      topologySpreadConstraints:
      - maxSkew: 1
        topologyKey: "eks.topology"
        whenUnsatisfiable: ScheduleAnyway
        labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - my-app
      containers:
      - name: my-container
        image: nginx
        ports:
        - containerPort: 80
EOF
kubectl apply -f ptsc.yaml
```

Now, you can see that the 6 pods were spread equally across the 3 topology keys: the Boston Wavelength Zone, NYC Wavelength Zone and the parent region.
```
NAME                     READY   STATUS    RESTARTS   AGE   IP            NODE                          NOMINATED NODE   READINESS GATES
nginx-7dbc97f869-9mcs9   1/1     Running   0          4s    10.0.2.212    ip-10-0-2-160.ec2.internal    <none>           <none>
nginx-7dbc97f869-ds8q8   1/1     Running   0          4s    10.0.11.218   ip-10-0-11-124.ec2.internal   <none>           <none>
nginx-7dbc97f869-f42tz   1/1     Running   0          4s    10.0.10.183   ip-10-0-10-48.ec2.internal    <none>           <none>
nginx-7dbc97f869-kfck6   1/1     Running   0          4s    10.0.11.64    ip-10-0-11-124.ec2.internal   <none>           <none>
nginx-7dbc97f869-z2pxd   1/1     Running   0          4s    10.0.10.228   ip-10-0-10-48.ec2.internal    <none>           <none>
nginx-7dbc97f869-z6fnx   1/1     Running   0          4s    10.0.2.65     ip-10-0-2-160.ec2.internal    <none>           <none>
````

Using this approach, Pod Topology Spread Constraints can be a highly valuable approach to distribute workloads across available Wavelength Zones in a more perscriptive way, beyond DaemonSet objects.