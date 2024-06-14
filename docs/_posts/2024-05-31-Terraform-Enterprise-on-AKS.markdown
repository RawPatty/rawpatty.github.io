---
layout: post
title:  "Setting up Terraform Enterprise in a new private AKS cluster with the new Flexible Deployment Option"
date:   2024-05-31 11:40:00 +0000
---
After a long hiatus I'm back with another writeup!

Recently I went through the journey of setting up Terraform Enterprise for my organisation - as there was bit of a lack of documentation on the step by step - Hashicorp is writing the documentation from the point of view that you already have the private AKS cluster running in your organisation, I decided to write a guide that captures the steps required to set this up from scratch.

I'll be covering the private AKS Active-Active setup using the flexible deployment option

## Prerequisties

Before getting started you will need a few things

- A Terraform Enterprise license
- Access to a virtual network with some subnets
- Contributor access to an Azure subscription


## Resouce Deployment
# Networking 

For most of these Azure resources it's quite straightforward - the only thing to note is that running all of these resources with a private link will require the associated zone and appropriate DNS fowarders set up (Depending on where how your DNS is resolved)
[Azure Private Endpoint DNS Zone value reference]

For this deployment you will need private DNS zones set up for 

- Azure Database for PostgresSQL flexible server (privatelink.postgres.database.azure.com)
- Azure Kubernetes services (privatelink.YOURREGION.azmk8s.io)
- Azure Cache for Redis (privatelink.redis.cache.windows.net)
- Azure Storage Account - Blob storage service (privatelink.blob.core.windows.net)
- Create a virtual network (if required to peer back to a hub, ensure the address space does not overlap your peered network)
- Create subnets - I set up three which I think would be the minimum, 
(/26 for AKS cluster, /28 for postgressql, /27 for private endpoints, apply the NSG and Route tables to the subnets)

# Infrastructure
- Create Postgres SQL Flexible server, deployed into the subnet (This creates a subnet delegation, which means the subnet cannot be used for other services as it is now managed by Azure)
- Create Azure Cache for Redis (with private endpoint)
- Create Storage account (with private endpoint) and blob container

# AKS Private cluster 
- Create user assigned managed identity
- Assign network contributor role on virtual network
- Assign Private DNS Zone Contributor on the privatelink.YOURZONE.azmk8s.io zone (Replace westus2 with desired region)
- Create Private AKS cluster with following az cli command exampl
{% highlight ruby %}
az aks create \
    --name "AKS Cluster Name" \
    --resource-group "Resource Group" \
    --load-balancer-sku standard \
    --enable-private-cluster \
    --assign-identity "Your user assigned managed identity ResourceID" \
    --private-dns-zone "Your Private DNS zone ResourceID" \
    --generate-ssh-keys \
    --vnet-subnet-id "AKS subnet resourceID" \
    --zones 1 2 3 \
    --node-resource-group "Choose your node resource group name"
{% endhighlight %}

# Configuration 
- Initialize the AKS cluster by adding in the desired node pool configurations, update schedules, diagnostics, and namespaces that weren't included in your script
- Initialize the PGSQL Database 
In `Server Parameters` on the Azure portal, find azure.extension and from the dropdown, add `CITEXT`, `HSTORE`, and `UUID-OSSP` extensions, click save.

Connecting to the PGSQL instance with the default account in a terminal
Create the new database - In my case I called it `terraform_enterprise`

Grant access for the terraform user account to have full control over schemas
{% highlight ruby %}
GRANT ALL PRIVILEGES ON DATABASE "DB_Name" TO "TF_Username";
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO "TF_Username";
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO "TF_Username";
GRANT ALL ON SCHEMA public TO "TF_Username";
{% endhighlight %}

# Deploy Terraform
In a local terminal, you will need to autheticate and grab the Terraform Enterprise Image. 


Log in to the Terraform Enterprise container image registry.
The `UserName` is `Terraform` and password is your license file value. (The below script will save you from entering it manually)

{% highlight ruby %}
cat <PATH_TO_HASHICORP_LICENSE_FILE> |  docker login --username terraform images.releases.hashicorp.com --password-stdin
{% endhighlight %}

Pull the Terraform Enterprise image from the registry as a test.

{% highlight ruby %}
docker pull images.releases.hashicorp.com/hashicorp/terraform-enterprise:<vYYYYMM-#>
{% endhighlight %}

Create your AKS namespace 
Create an secret within your AKS for accessing the image 

{% highlight ruby %}
kubectl create secret docker-registry terraform-enterprise --docker-server=<DOCKER_REGISTRY_URL> --docker-username=<DOCKER_REGISTRY_USERNAME> --docker-password=<DOCKER_REGISTRY_PASSWORD>  -n <TFE_NAMESPACE>
{% endhighlight %}

A copy of the values file can be obtained at [Hashicorp Terraform Enterprise Github Page]

Reference for the values can be found at [Configuration Reference]

Once that is filled out you are ready to kick off the install!

{% highlight ruby %}
helm install terraform-enterprise hashicorp/terraform-enterprise --namespace <YOUR_NAMESPACE> -f "Pathtovalues.yaml"
{% endhighlight %}


# Example Values file

Since I found it quite difficult to get all the right values, I'm supplying my own example one, hope it helps!

[Sample Values File]


## Reference Documentation

[Hashicorp Terraform Enterprise FDO Kubernetes install guide]

[Hashicorp Terraform Enterprise FDO Kubernetes install guide]: https://developer.hashicorp.com/terraform/enterprise/flexible-deployments/install/kubernetes/install
[Configuration Reference]: https://developer.hashicorp.com/terraform/enterprise/flexible-deployments/install/configuration
[Sample Values File]: https://gist.github.com/RawPatty/ec880f0962f534ea5866511f208ab4a3
[Hashicorp Terraform Enterprise Github Page]: https://github.com/hashicorp/terraform-enterprise-helm/blob/main/values.yaml 
[Azure Private Endpoint DNS Zone value reference]: https://learn.microsoft.com/en-us/azure/private-link/private-endpoint-dns
