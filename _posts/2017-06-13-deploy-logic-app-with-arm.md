---
layout: post
title: Deploy Logic Apps & API Connection with ARM
tags:
    - Azure
    - Logic Apps
---

One of the key question when developing Logic Apps is how to create an atomic re-deployable package that you will use to deploy your worfklow to another tenant. A typical use-case, is when you have distinct tenants for development and production. You need not only to deploy the workflow itself but also the API connections that you have created during the development.

Imagine you have this wonderfull Logic App that creates a blob in Azure Storage for each tasks you create in Wunderlist. 
<img src="/public/images/2017-06-logic-apps-deploy/logic-apps.png" width="250px" class="align-center" alt="Great Logic Apps"/>

When you try to export the ARM Template by clicking the Automation script button, then you get this message: 
<div class="notice--warning">
API Connections cannot be exported yet and is not included in the template. See error details.
</div>

If you try to deploy this template in another resource group or tenant, you will have to recreate the connections manually. This could be a huge task if you have a lot of Logic Apps and API Connections.
<img src="/public/images/2017-06-logic-apps-deploy/connections-missing.png" width="250px" class="align-center" alt="Great Logic Apps"/>

Hopefully, it is possible to create the API Connections automatically when you deploy your ARM Template.

### Creating API Connection

We will need to add a new resource into our ARM Template file. A basic deployment template for a connection looks like this:

```
{
    "type": "Microsoft.Web/connections",
    "apiVersion": "2016-06-01",
    "location": "[resourceGroup().location]",
    "name": "{ConnectionName}",
    "properties": {
        "api": {
            "id": "[concat('subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Web/locations/', resourceGroup().location, '/managedApis/{Api}')]"
        },
        "displayName": "{DisplayName}",
        "parameterValues": { 
            "{ParameterValues}": "{ParameterValues}"
        }
    }
}
```

From this you have to replace the following token:

|Token|Description|
|---|---|
|{ConnectionName}|The technical name of your connection|
|{Api}|The api kind. In our case 'wunderlist'|
|{DisplayName}|The display name of you connection|
|{ParameterValues}|A key/value list of parameters|

### Retrieve the API Parameters

The {ParameterValues} are dependent of the api kind. The question is how to retrieve the needed parameters for a given API? I personally use [armclient](https://github.com/projectkudu/ARMClient) to get the information metadata.

```
armclient.exe get https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Web/locations/{region}/managedApis/{Api}?api-version=2016-06-01
```

From the response, you could see the parameters that you need for your deployment. 
In our case, for the Azure Storage it is: ```accountName``` and ```accessKey```.

```
{
    "properties": {                                                                          
        "name": "azureblob",                                                                   
        "connectionParameters": {                                                              
            "accountName": {                                                                     
                "type": "string",                                                                  
                "uiDefinition": {                                                                  
                "displayName": "Azure Storage Account name",                                     
                "description": "Name of the storage account the connector should use.",          
                "tooltip": "Provide the storage account name",                                   
                "constraints": {                                                                 
                    "required": "true"                                                             
                    }                                                                                
                }                                                                                  
            },                                                                                   
            "accessKey": {                                                                       
                "type": "securestring",                                                            
                "uiDefinition": {                                                                  
                "displayName": "Azure Storage Account Access Key",                               
                "description": "Specify a valid primary/secondary storage account access key.",  
                "tooltip": "Specify a valid primary/secondary storage account access key.",      
                "constraints": {                                                                 
                    "required": "true"                                                             
                    }                                                                                   
                }                                                                                  
            }                                                                                    
        }                                                                                     
        // Cut for brevity
    }
}
```

So we can add our connection resource to the ARM Template, we will start by creating 2 parameters to make our template re-usable and add the API connections into the resources array. 

```
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "workflows_test_name": {
            "defaultValue": "test",
            "type": "string"
        },
        "storage_accountName": {
            "type": "string"
        },
        "storage_accessKey": {
            "type": "securestring"
        }
    },
    "resources": [
        {
            "type": "Microsoft.Web/connections",
            "apiVersion": "2016-06-01",
            "location": "[resourceGroup().location]",
            "name": "azureblob",
            "properties": {
                "api": {
                    "id": "[concat('subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Web/locations/', resourceGroup().location, '/managedApis/azureblob')]"
                },
                "displayName": "test blob",
                "parameterValues": {
                    "accountName": "[parameters('storage_accountName')]",
                    "accessKey": "[parameters('storage_accessKey')]"
                }
            }
        },
        {
            "type": "Microsoft.Web/connections",
            "apiVersion": "2016-06-01",
            "location": "[resourceGroup().location]",
            "name": "wunderlist",
            "properties": {
                "api": {
                    "id": "[concat('subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Web/locations/', resourceGroup().location, '/managedApis/wunderlist')]"
                },
                "displayName": "test list",
                "parameterValues": {
                }
            }
        }
        // Cut for brevity
    }
}
```

Note, that we did not add any content to the ```parameterValues``` for the Wunderlist connection because it is an OAuth connection. We will then add these 2 new resources as dependencies on ```dependsOn``` for the Logic Apps and reference them on the ```$connections```.

```
{
    "resources": [
        // Cut for brevity
        {
            "type": "Microsoft.Logic/workflows",
            "name": "[parameters('workflows_test_name')]",
            "apiVersion": "2016-06-01",
            // Cut for brevity
            "properties": {
                // Cut for brevity
                "parameters": {
                    "$connections": {
                        "value": {
                            "azureblob": {
                                "connectionId": "[resourceId('Microsoft.Web/connections', 'azureblob')]",
                                "connectionName": "azureblob",
                                "id": "[reference(concat('Microsoft.Web/connections/', 'azureblob'), '2016-06-01').api.id]"
                            },
                            "wunderlist_1": {
                                "connectionId": "[resourceId('Microsoft.Web/connections', 'wunderlist')]",
                                "connectionName": "wunderlist",
                                "id": "[reference(concat('Microsoft.Web/connections/', 'wunderlist'), '2016-06-01').api.id]"
                            }
                        }
                    }
                }
            },
            "dependsOn": [
                "[resourceId('Microsoft.Web/connections', 'azureblob')]",
                "[resourceId('Microsoft.Web/connections', 'wunderlist')]"
            ]
        }
    ]
}
```

### Consent the OAuth connections

If you deploy now the ARM Template, you will see that both API Connections have been created. But when you open the Logic Apps, you will have to update manually the connection to Wunderlist by entering your credentials for the service.  

But if you need to finalize the API Connection creation without opening every Logic Apps then you can use this PowerShell script [LogicAppConnectionAuth](https://github.com/logicappsio/LogicAppConnectionAuth). This script will retrieve a consent link for a connection for an OAuth Logic Apps connector. It will then open the consent link and complete authorization to enable a connection. 

### What's next?

I hope this will help you with your Logic Apps deployment. The experience to create the API Connections for OAuth services is far from ideal. But for Azure resources it works perfectly. 