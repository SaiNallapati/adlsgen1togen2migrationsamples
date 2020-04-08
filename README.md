# Incremental Copy Pattern Guide : A quick start template

## Overview
The purpose of this document is to provide a manual for the Incremental copy pattern from Azure Data Lake Storage 1 (Gen1) to Azure Data Lake Storage 2 (Gen2) using Azure Data Factory and Powershell. As such it provides the directions, references, sample code examples of the PowerShell functions been used. It is intended to be used in form of steps to follow to implement the solution from local machine.
This guide covers the following tasks:

   * Set up for Incremental copy pattern from Gen1 to Gen2 

   * Enumerating Gen1 and Gen2 data into CSV

   * Data Validation and Comparison between Gen1 and Gen2 data using CSV

### Prerequisites 
You need below:

* An Azure account with an active subscription 

* Resource group 

* Azure Storage account with Data Lake Storage Gen1. For more details please refer to [create azure data lake storage for Gen1](https://docs.microsoft.com/en-us/azure/data-lake-store/data-lake-store-get-started-portal).

 * Azure Storage account with Data Lake Storage Gen2.For more details please refer to [create azure storage account](https://docs.microsoft.com/en-us/azure/storage/common/storage-account-create?tabs=azure-portal) 

* Service principal with permission on the subscription. To learn more see [create service principal account](https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-create-service-principal-portal) and to provide SPN access to Gen1 refer to [SPN access to Gen1](https://docs.microsoft.com/en-us/azure/data-lake-store/data-lake-store-service-to-service-authenticate-using-active-directory)

* Azure Data Factory(v2) 

* Windows Powershell ISE.

   ##### Install below modules :
   
   * PowerShellGetRepository PSGallery -Force 

   * Azure storage - version 1.13.3-preview 
   
    ```scala
    // Below is the command   
       
      Install-Module az.storage -RequiredVersion 1.13.3-preview -Repository PSGallery -AllowClobber -AllowPrerelease -Force
   
   ```
    * Az.DataLakeStore . Refer to [Import module Az Datalakestore](https://docs.microsoft.com/en-us/powershell/module/az.datalakestore/import-azdatalakestoreitem?view=azps-3.7.0)

## Steps to be followed

### 1. Migration Pipeline Setup
This step will ensure that the configuration file is ready before running the azure data factory pipeline for incremental copy pattern. 
The config file sample format is available on GitHub in [config file sample](https://github.com/rukmani-msft/adlsgen1togen2migrationsamples/tree/develop/Src/Migration/).

#### Download the repo to your local machine :
![image](https://user-images.githubusercontent.com/62353482/78593702-e4f54f80-77fb-11ea-8bfb-2ecc8e8ed757.png) 

 **Note** : To avoid security warning error --> Open the zip folder , right click and Goto properties --> General --> Check unblock option under security section.

The downloaded migration folder will contain below listed contents :

![image](https://user-images.githubusercontent.com/62351942/78715961-02491d00-78d3-11ea-89e5-5132cf49898d.png)

### Glossary on Contents 

**1.DataFactoryV2Template** : This folder contain all the json templates which is being used for creating dynamic azure data factory.

**2.InventoryInput.json** : This config file contains all the details of gen1 and gen2 ADLS. In this we have to list all the source and destination folders,which is being used to create data factory pipeline activities.Config pipeline elements contain number of pipelines to be created. We can have one time full load pipelines and incremental pipelines.

**Note** : Setting multiple pipelines and activities enables parallelism mechanism.

**3.InvokeMethod.ps1**: This powershell script will execute PipelineConfig.ps1 and DataFactory.ps1

**4.PipelineConfig.ps1** : This powershell script will create all the required json input data, which is being used in Datafactory.ps1 powershell.This will dynamically create the json file considering all the required inputs from InventoryInput.json file.

**5.DataFactory.ps1** : This powershell will create the linked services, datasets and pipeline in sequence order,based on the input provided in InventoryInput.json

**Path for config file** : [InventoryInput.json](https://github.com/rukmani-msft/adlsgen1togen2migrationsamples/blob/develop/Src/Migration/InventoryInputs.json)

```scala

//Below is the code snapshot for setting the configuration file for each variable

{
  "gen1SourceRootPath" : "https://<<adlsgen1>>.azuredatalakestore.net/webhdfs/v1", // Provide the source Gen1 root path 
  "gen2SourceRootPath" : "https://<<adlsgen2>>.dfs.core.windows.net", // Provide the Gen2 source root path
  "tenantId" : "<<tenantId>>", // Provide the tenantId 
  "subscriptionId" : "<<subscriptionId>>", // Provide the SubscriptionId 
  "servicePrincipleId" : "<<servicePrincipleId>>", // Provide the servicePrincipleId
  "servicePrincipleSecret" : "<<servicePrincipleSecret>>", // Provide the servicePrinciplesecret key 
  "factoryName" : "<<factoryName>>", // Give the factory name e.g Gen1ToGen2DataFactory 
  "resourceGroupName" : "<<resourceGroupName>>", // Give the resource group name 
  "location" : "<<location>>", // Provide the Data factory location 
  "overwrite" : "true", // default 

```

**Setting up and scheduling the Factory pipeline for Incremental copy pattern**

```scala

//Below is how to configure and schedule the data factory pipeline 

 "pipeline": [  
	{
		"pipelineId" : "1",   // Set this value between 1 and 50 to start factory and run in parallel  
		"isChurningOrIsIncremental" : "true",   // Value is set to true for Incremental copy pattern
		"triggerFrequency" : "Minute",   // frequency in units 
		"triggerInterval" : "15",   // Set the time interval for scheduling 
		"triggerUTCStartTime" : "2020-04-07T13:00:00Z",   // Provide the UTC time to start the factory 
		"pipelineDetails":[			
			{			
				"sourcePath" : "/AdventureWorks/RawDataFolder/Increment/FactFinance",  // Give the Gen1 source path from the 
				"destinationPath" : "AdventureWorks/RawDataFolder/Increment/FactFinance",
				"destinationContainer" : "gen1sample"
			},
			{			
				"sourcePath" : "/AdventureWorks/RawDataFolder/Increment/FactInternetSales",
				"destinationPath" : "AdventureWorks/RawDataFolder/Increment/FactInternetSales",
				"destinationContainer" : "gen1sample"
```

### 2. Post Migration Checks 

:heavy_check_mark: Check the data factory pipeline creation in ADF site 

:heavy_check_mark: Data in forms of files and folders landed to Gen2 path.

### 3. Data Validation

This step ensures that the right data is migrated from Gen1 to Gen2.To validate the same process ,below is the sequence of functions being called out in a single script :

#### 3.1 InvokeValidation 
#### 3.2 GetGen1Inventory
#### 3.3 GetGen2Inventory
#### 3.4 CompareGen1andGen2

Run the script (##name##) which will read the Gen1Inventory and Gen2Inventory 



### 4. Comparison Report


### 5. Application Migration check 

**5.1** Get the mount path for Gen1 

**5.2** Decommission Gen1 load 

**5.3** Change and configure the mount path to Gen2 storage 

**5.4** Re schedule the migration pipeline as per above path 


## Reach out to us

### You found a bug or want to propose a feature?

File an issue here on GitHub: [![File an issue](https://img.shields.io/badge/-Create%20Issue-6cc644.svg?logo=github&maxAge=31557600)](https://github.com/rukmani-msft/adlsgen1togen2migrationsamples/issues/new).
Make sure to remove any credential from your code before sharing it.

### References

[Azure Data Lake Storage migration from Gen1 to Gen2 ](https://docs.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-migrate-gen1-to-gen2)

