# Pulumi Demo Project

This project uses the open source bits of the Pulumi IaC tool for my personal
learning and development. Generally, I document my work from the perspective
of running it as IaC using service accounts. You will find that most online
quickstarts tend to use your personal credentials. This is a bad practice if 
you indend to use your code in a pipeline.  

## Getting Started
* Quickstart: <https://www.pulumi.com/docs/clouds/azure/get-started/begin/>
* Linux non-root install - `curl -fsSL https://get.pulumi.com | sh` 

## Random useful commands
* Some commands used are embedded below in my example env file.
* Log into your local sessions as your service account: 
```bash
az login \
--service-principal \
--username ${ARM_CLIENT_ID} \ 
--password ${ARM_CLIENT_SECRET} \ 
--tenant ${ARM_TENANT_ID} \
--output table`
```

## Example env file to source, YMMV
```bash
#!/usr/bin/env bash

### bootstrap commands
### az group create --name "rg-example-bootstrap" --location "EastUS"
### az storage account create --name "pulumistorageaccount" --resource-group "rg-example-bootstrap" --location "EastUS" --sku "Standard_LRS"
### az storage container create -n "container-pulumi" --account-name "pulumistorageaccount"

azure_resource_group="rg-example-bootstrap"
azure_storage_account="pulumistorageaccount"
azure_storage_container="container-pulumi"

### needed service principal values
### "az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/<your_subscription>" --years 5"

export ARM_CLIENT_ID="<appId>"
export ARM_CLIENT_SECRET="<password>"
export ARM_TENANT_ID="<tenant>"
export ARM_SUBSCRIPTION_ID="<subsciption>"

### pulumi specific values
export PULUMI_BACKEND_URL="azblob://${azure_storage_account}.blob.core.windows.net/${azure_storage_container}"
export AZURE_STORAGE_ACCOUNT=${azure_storage_account}
export AZURE_STORAGE_KEY=$(az storage account keys list --resource-group ${azure_resource_group} --account-name ${azure_storage_account} --query "[0].value" --output tsv)
```

## Todo
* Refactor SP command to dump the results to a keyvault and pull ARM values
dynamically. This should work like the AZURE_STORAGE_KEY value. This would make
future SP secret rotation dynamic.
