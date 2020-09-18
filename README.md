# Bright Sessions: AKS Provisioning en Configuration met Terraform
Welcome to the repo for the "AKS Provisioning en Configuration met Terraform" Bright Sessions webinar hosted by Bright Cubes' Tommy de Jong on 16-09-2020.
Check out the presentation (in Dutch) here: https://www.youtube.com/watch?v=zTje8b1stGo

## Tools used
The code serves as an example to deploy an AKS cluster and serve a Hello World application over HTTPS. To achieve this, the following tools are used:
* **Terraform** to deploy the infrastructure templates and configure the cluster (more details on this below)
* **Azure Resource Manager (ARM)** template to demonstrate combining Terraform & ARM deploy the AKS cluster. The included ARM template will deploy a cluster with two node pools with autoscaling enabled (max. 2 instances), running on ``Standard_B2ms`` virtual machines.
* **Helm** to deploy the **NGINX Kubernetes Ingress Controller** and **Cert-Manager** charts for routing external traffic securely using HTTPS to the Hello World application
* **Kubernetes** to run the Hello World application using a Deployment, Service and Ingress.

## Prerequisites
You need Terraform => v0.13.0, kubectl => v1.18, Azure CLI => v2.10.0 and Helm => v3.0. 
You also need an Azure subscription where you have permissions to create resource groups and service principals. You need to be logged in to the Azure subscription using ``az login``. Terraform can use these credentials.
The demo relies on an existing Azure Virtual Network (VNet) called ``webinar-vnet`` in an existing Resource Group called ``webinar-rg``. You can create these yourself, adapt the script to have Terraform create this as practice, or change the values in the code.
Also be sure to enter your e-mail address to the ``ClusterIssuer`` resource in ``.\apps\manifests\clusterissuer.yaml``.
If you change any of the values with regards to the domain name, be sure to change this in the deployment YAML in ``.\apps\manifests\aks-helloworld.yaml``.

## Notes and limitations
This code serves as an example to demonstrate some key features of Terraform (such as ``local`` ``data`` and ``modules``) and integrations with other tools, which was the main goal of the webinar. 
It does not necessarily follow best practices. For example, the code is split into two parts: one part to initialize the infrastructure in Azure and a second part to configure the cluster and deploy the applications. Outputs from one part need to be manually copied as inputs for the second part, which is quite cumbersome. Also, the Kube config needs to be added manually and some steps require local execution.
Ideally the two parts should be combined in a single script, and be prepared to run in a DevOps pipeline. 

However, for the sake of the demo this was done in this way, also to demonstrate that Terraform can be used as a *configuration management* tool, similar to Ansible, even though this is not best practice.

## Get started
1. To get started and run this code, you first need to change some values to match your environment. Variables and derivations are found in ``.\variables.tf`` or the ``locals`` block in ``.\main.tf``. Some of the variables have default values that can be overridden in ``variables.tf`` or when running ``terraform apply``. If needed, change the values for the existing VNet in the ``data`` block in ``.\main.tf``. Verify these before running Terraform.
2. Run ``terraform init`` in the root folder to initialize Terraform and download the configured modules.
3. Run ``terraform apply`` (optionally by overriding default variable values appending ``-var <key=value>``), verify the changes and resource names and confirm resource creation.
   Terraform will create a ``resource group, subnet, service principal, AKS cluster & public IP address`` using the variables provided. Creation of all these resources will take around 5-10 minutes.
4. Three values will be outputted, that need to serve as input for the second part for cluster configuration: an IP address, DNS label and Resource Group name. The fourth output, the FQDN, is the URL where your Hello World application will be published. You do not need this as input for the second part.
5. Verify that your AKS cluster and additional resources were created by going to the Azure Portal. Enable ``kubectl`` access by going to your AKS cluster and clicking 'Connect' in the top bar, and run the ``az aks connect`` command. Verify that you are connected to the right cluster using ``kubectl get nodes`` and verify that you see your node pools.
6. Change to the ``apps`` directory and run ``terraform init`` again. Next, run ``terraform apply`` to create the necessary Kubernetes namespaces and deploy the Helm charts for NGINX and Cert-Manager. It will also deploy the required ``ClusterIssuer`` to enable HTTPS using Let's Encrypt. Terraform will wait for the Ingress controller to fully initialize before deploying the Hello World application.
7. When Terraform is finished, give Cert-Manager a minute or two to request and issue a certificate. You can check the progress using ``kubectl get events -w`` (the ``-w`` flag is used to stream the events). Once you see ``The certificate has been successfully issued``, go to your domain name from step 4 to verify that your application works and is secured by HTTPS!

If you have any questions about these instructions or the example provided here, contact Tommy de Jong at tommy.dejong@brightcubes.nl