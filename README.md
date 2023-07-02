# PrivateNodeEth
How to set up a private eth node on Azure containers

### Basic Azure Commands

https://github.com/ferhaty/azure-cli-cheatsheet


#### Setting up environment

1. login into private azure
```az login```
2. Set up environment variables used in various commands to follow.
```
RESOURCE_GROUP="blockchain-apps-group"
ENVIRONMENT_NAME="blockchain-storage-environment"
LOCATION="southeastasia"
```
3. Ensure you have the latest version of the Container Apps Azure CLI extension
```
az extension add -n containerapp --upgrade
```
4. Register the Microsoft.App namespace.
```
az provider register --namespace Microsoft.App
```
5. Register the Microsoft.OperationalInsights provider for the Azure Monitor Log Analytics workspace if you haven't used it before.
```
az provider register --namespace Microsoft.OperationalInsights
```

#### Create an environment
1. Create resource group, can delete all resource after
```
az group create \
  --name $RESOURCE_GROUP \
  --location $LOCATION \
  --query "properties.provisioningState"
```
2. Create a Container Apps environment.
```
az containerapp env create \
  --name $ENVIRONMENT_NAME \
  --resource-group $RESOURCE_GROUP \
  --location "$LOCATION" \
  --query "properties.provisioningState"
```

#### Set up a storage account
1. Define a storage account name.
```
STORAGE_ACCOUNT_NAME="blockchainstorageaccount$RANDOM"
```
2. Create an Azure Storage account.
```
az storage account create \
  --resource-group $RESOURCE_GROUP \
  --name $STORAGE_ACCOUNT_NAME \
  --location "$LOCATION" \
  --kind StorageV2 \
  --sku Standard_LRS \
  --enable-large-file-share \
  --query provisioningState
```
3. Define a file share name and create it
```
STORAGE_SHARE_NAME="Ethereum"

az storage share-rm create \
  --resource-group $RESOURCE_GROUP \
  --storage-account $STORAGE_ACCOUNT_NAME \
  --name $STORAGE_SHARE_NAME \
  --quota 1024 \
  --enabled-protocols SMB \
  --output table
```

4. Get the storage account key.
```
STORAGE_ACCOUNT_KEY=`az storage account keys list -n $STORAGE_ACCOUNT_NAME --query "[0].value" -o tsv`
```

5. Define the storage mount name.
```
STORAGE_MOUNT_NAME="myblockchaindatamount"
```

#### Create the storage mount
1. Create the storage link in the environment.
```
az containerapp env storage set \
  --access-mode ReadWrite \
  --azure-file-account-name $STORAGE_ACCOUNT_NAME \
  --azure-file-account-key $STORAGE_ACCOUNT_KEY \
  --azure-file-share-name $STORAGE_SHARE_NAME \
  --storage-name $STORAGE_MOUNT_NAME \
  --name $ENVIRONMENT_NAME \
  --resource-group $RESOURCE_GROUP \
  --output table
```

2. 


