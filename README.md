# Pulumi Demo Project | Azure Backend
This project uses the open source bits of the Pulumi IaC tool for my personal
learning and development. Generally, I document my work from the perspective
of running it as IaC using service accounts. You will find that most online
quickstarts tend to use your personal credentials. This is a bad practice if 
you indend to use your code in a pipeline. Other backend are supported. I am
starting with Azure.

## References
[Pulumi Quickstart Guide](<https://www.pulumi.com/docs/clouds/azure/get-started/begin/>) 

## Getting Started
 
```bash
# Pulumi Install
curl -fsSL https://get.pulumi.com | sh
``` 

```bash
# Azure CLI install, trust but verify
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

## Steps
Be mindful that there are two phases to bootstrapping a new Pulumi project in
Azure. The first would be the **bootstrap** steps. You will perform these as
your root Azure personal account.

```bash
# Azure login stuff
az login --use-device-code   
az account list --output table 
az account set --subscription <your_subscription>
``` 

```bash
# Create your resource group, storage account, and container
# The Azure storage account name needs to be globally unique
az group create --name "rg-example-bootstrap" --location "EastUS"  
az storage account create --name "pulumi001storageaccount" --resource-group "rg-example-bootstrap" --location "EastUS" --sku "Standard_LRS"
az storage container create -n "pulumi-backend" --account-name "pulumi001storageaccount" --auth-mode login
```  

Create your Azure service principal for future automation and local dev. The
outputs from this command need to be added to your local environment file as
the `ARM_` variables listed in the example env file section below. These are
secrets. **Do not commit these values.** Once your env file is created, `source`
it and the login variable values will be available for the next command's
success.
```bash
az ad sp create-for-rbac \
--role="Contributor" \
--scopes="/subscriptions/382fdf5d-5bdf-4cba-9946-7637db686413" \
--years 5
```

Login locally as your newly created service principal.
```bash
az login \
--service-principal \
--username ${ARM_CLIENT_ID} \ 
--password ${ARM_CLIENT_SECRET} \ 
--tenant ${ARM_TENANT_ID} \
--output table
```

## Login to your new Pulumi backend on Azure!
```bash
pulumi login azblob://pulumi-backend
```

## Example env file to source, YMMV
This file is mean to be created for local use only. I usually keep it in a
`/tmp` folder which is included in my `.gitignore`. The idea being, you can
just `source` it and be up and running on local dev. For pipeline work, you
would add the environment variables to your CI platform of choice.

```bash
#!/usr/bin/env bash

### Azure resources
azure_resource_group="rg-example-bootstrap"
azure_storage_account="pulumi001storageaccount"
azure_storage_container="pulumi-backend"

### Service principal values
export ARM_CLIENT_ID="<appId>"
export ARM_CLIENT_SECRET="<password>"
export ARM_TENANT_ID="<tenant>"
export ARM_SUBSCRIPTION_ID="<subsciption>"

### Pulumi values
export PULUMI_BACKEND_URL="azblob://${azure_storage_account}.blob.core.windows.net/${azure_storage_container}"
export AZURE_STORAGE_ACCOUNT=${azure_storage_account}
export AZURE_STORAGE_KEY=$(az storage account keys list --resource-group ${azure_resource_group} --account-name ${azure_storage_account} --query "[0].value" --output tsv)
```

## Todo
* Refactor SP command to dump the results to a keyvault and pull ARM values
dynamically. This should work like the AZURE_STORAGE_KEY value. This would make
future SP secret rotation dynamic.
