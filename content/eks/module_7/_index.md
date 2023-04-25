+++
title = "Module 7 - Observability and Performance Testing"
weight = 60
+++

After deploying the model successfully, we may want to monitor to performance of the underlying infrastructure in which your model is hosted, the application itself, or the health of the EKS cluster itself. To do so, the AWS Observability Accelerator offers a one-click solution that deploys AWS Distro for OpenTelemetry collector to collect metrics, stores the metrics on Amazon Managed Service for Prometheus and configure out-of-the-box Amazon Managed Grafana dashboards.

To get started, clone this GitHub repo, navigate to the existing-cluster-with-base-and-infra sub-directory, and edit the terraform.tfvars with your environment variables. As an example, from the Terraform module above, your cluster name may be eks-test-Cluster. Lastly, run terraform apply to create the infrastructure required for the AWS Observability Accelerator.

```
 git clone https://github.com/aws-observability/terraform-aws-observability-accelerator.git
 
 cd terraform-aws-observability-accelerator/examples/existing-cluster-with-base-and-infra
  
 cat << EOF > terraform.tfvars
 
 # (mandatory) AWS Region where your resources will be located
  aws_region = ""
  
 # (mandatory) EKS Cluster name
  eks_cluster_id = ""
  
 # (mandatory) Amazon Managed Grafana Workspace ID: ex: g-abc123
  managed_grafana_workspace_id = ""
 
 # (optional) Leave it empty for a new workspace to be created
 # managed_prometheus_workspace_id = ""
  
 # (mandatory) Grafana API Key - https://docs.aws.amazon.com/grafana/latest/userguide/API_key_console.html
  grafana_api_key = ""
 
 EOF
 terraform init
 terraform apply -auto-approve # Be sure to Save & Test Data Sources by navigating to https://<your-workspace>.<region>.amazonaws.com/datasources/edit/
```
After a few minutes, you should see the ADOT collector and Prometheus by running `kubectl get pods --all-namespaces`. 

At this point, navigate to your Amazon Managed Grafana workspace and browse the available dashboads!

![Amazon Managed Service for Grafana Dashboards](./eks/module_7/amg.png)