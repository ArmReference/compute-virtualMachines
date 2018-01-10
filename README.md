# compute-virtualMachines
Reference Template
```
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "OperatingSystem": {
      "type": "string",
      "defaultValue": "Windows Server 2012 R2",
      "allowedValues": [
        "Windows Server 2012 R2",
        "Windows Server 2016 DataCenter"
      ]
    },
    "AdminUsername": {
      "type": "string",
      "defaultValue": "adminAccount"
    },
    "AdminPassword": {
      "type": "securestring",
      "defaultValue": "P@ssw0rd1234"
    },
    "Size": {
      "type": "string",
      "defaultValue": "Ds1_V2",
      "allowedValues": [
        "A1",
        "D1_V2",
        "D2_V2",
        "D11_V2",
        "D12_V2",
        "D13_V2",
        "D14_V2",
        "D15_V2",
        "F2",
        "F4",
        "F8",
        "F16",
        "Ds1_V2",
        "Ds2_V2",
        "Ds11_V2",
        "Ds12_V2",
        "Ds13_V2",
        "Ds14_V2",
        "Ds15_V2",
        "F2s",
        "F4s",
        "F8s",
        "F16s"
      ]
    },
    "NetworkName": {
      "type": "string"
    },
    "NetworkResourceGroupName": {
      "type": "string"
    },
    "SubnetName": {
      "type": "string"
    },
    "TimeZone": {
      "type": "string",
      "allowedValues": [
        "Pacific Standard Time",
        "Mountain Standard Time",
        "Central Standard Time",
        "Eastern Standard Time"
      ]
    },
    "StorageAccountName": {
      "type": "string"
    },
    "ContainerName": {
      "type": "string"
    },
    "SasToken": {
      "type": "string"
    }
  },
  "variables": {
    "computeUri": "[concat('https://',parameters('StorageAccountName'),'.blob.core.windows.net/',parameters('ContainerName'),'/Microsoft.Compute/virtualMachines')]",
    "availabilitySetUri": "[concat('https://',parameters('StorageAccountName'),'.blob.core.windows.net/',parameters('ContainerName'),'/Microsoft.Compute/availabilitySets')]",
    "disksUri": "[concat('https://',parameters('StorageAccountName'),'.blob.core.windows.net/',parameters('ContainerName'),'/Microsoft.Compute/disks')]",
    "StorageUri": "[concat('https://',parameters('StorageAccountName'),'.blob.core.windows.net/',parameters('ContainerName'),'/Microsoft.Storage/storageAccounts')]",
    "interfacesUri": "[concat('https://',parameters('StorageAccountName'),'.blob.core.windows.net/',parameters('ContainerName'),'/Microsoft.Network/networkInterfaces')]",
    "OS": {
      "Windows Server 2012 R2": {
        "imagePublisher": "MicrosoftWindowsServer",
        "imageOffer": "WindowsServer",
        "imageSKU": "2012-R2-Datacenter",
        "version": "latest"
      },
      "Windows Server 2016 DataCenter": {
        "imagePublisher": "MicrosoftWindowsServer",
        "imageOffer": "WindowsServer",
        "imageSKU": "2016-Datacenter",
        "version": "latest"
      }
    },
    "avSetName": "builtfromvars",
    "interfaceName": "builtfromvars",
    "vnetId": "[resourceId(parameters('NetworkResourceGroupName'), 'Microsoft.Network/virtualNetworks',parameters('NetworkName'))]",
    "subnetId": "[concat(variables('vnetId'),'/subnets/',parameters('SubnetName'))]",
    "computerName": "builtfromvars",
    "timeZone": "[parameters('TimeZone')]",
    "vmSize": "[concat('Standard_',parameters('Size'))]",
    "DiagnosticsStorageAccount": "[concat(uniquestring(resourceGroup().id), 'standardsa')]"
  },
  "resources": [
    {
      "name": "BuildVirtualMachine",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2016-09-01",
      "dependsOn": [
        "[concat('Microsoft.Resources/deployments/','BuildProperties')]"
      ],
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[concat(variables('computeUri'), '/virtualMachines.json', parameters('SasToken'))]",
          "contentVersion": "2017.09.01.0"
        },
        "parameters": {
          "name": {
            "value": "[variables('computerName')]"
          },
          "tags": {
            "value": "[json('{\"TagName\": \"TagValue\"}')]"
          },
          "plan": {
            "value": { }
          },
          "properties": {
            "value": "[reference('BuildProperties').outputs.properties.value]"
          }
        }
      }
    },
    {
      "name": "BuildProperties",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2016-09-01",
      "dependsOn": [
        "[concat('Microsoft.Resources/deployments/','BuildAvailabilitySet')]",
        "[concat('Microsoft.Resources/deployments/','BuildDiagnosticsProfile')]",
        "[concat('Microsoft.Resources/deployments/','BuildHardwareProfile')]",
        "[concat('Microsoft.Resources/deployments/','BuildNetworkProfile')]",
        "[concat('Microsoft.Resources/deployments/','BuildOsProfile')]",
        "[concat('Microsoft.Resources/deployments/','BuildStorageProfile')]"
      ],
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[concat(variables('computeUri'), '/properties.json', parameters('SasToken'))]",
          "contentVersion": "2017.09.01.0"
        },
        "parameters": {
          "licenseType": {
            "value": ""
          },
          "availabilitySet": {
            "value": "[reference('BuildAvailabilitySet').outputs.availabilitySet.value]"
          },
          "hardwareProfile": {
            "value": "[reference('BuildHardwareProfile').outputs.hardwareProfile.value]"
          },
          "storageProfile": {
            "value": "[reference('BuildStorageProfile').outputs.storageProfile.value]"
          },
          "osProfile": {
            "value": "[reference('BuildOsProfile').outputs.osProfile.value]"
          },
          "networkProfile": {
            "value": "[reference('BuildNetworkProfile').outputs.networkProfile.value]"
          },
          "diagnosticsProfile": {
            "value": "[reference('BuildDiagnosticsProfile').outputs.diagnosticsProfile.value]"
          }
        }
      }
    },
    {
      "name": "DeployAvailabilitySet",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2016-09-01",
      "dependsOn": [ ],
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[concat(variables('availabilitySetUri'), '/availabilitySets.json', parameters('SasToken'))]",
          "contentVersion": "2017.09.01.0"
        },
        "parameters": {
          "name": {
            "value": "[variables('avSetName')]"
          },
          "skuName": {
            "value": "Aligned"
          },
          "tags": {
            "value": "[json('{\"TagName\": \"TagValue\"}')]"
          }
        }
      }
    },
    {
      "name": "BuildAvailabilitySet",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2016-09-01",
      "dependsOn": [
        "[concat('Microsoft.Resources/deployments/','DeployAvailabilitySet')]"
      ],
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[concat(variables('computeUri'), '/availabilitySet.json', parameters('SasToken'))]",
          "contentVersion": "2017.09.01.0"
        },
        "parameters": {
          "id": {
            "value": "[resourceId(resourceGroup().name,'Microsoft.Compute/availabilitySets',variables('avSetName'))]"
          }
        }
      }
    },
    {
      "name": "BuildIpConfigurations",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2016-09-01",
      "dependsOn": [ ],
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[concat(variables('interfacesUri'), '/ipConfigurations.json', parameters('SasToken'))]",
          "contentVersion": "2017.09.01.0"
        },
        "parameters": {
          "name": {
            "value": "[variables('interfaceName')]"
          },
          "subnet": {
            "value": "[variables('subnetId')]"
          }
        }
      }
    },
    {
      "name": "CreateNetworkInterface",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2016-09-01",
      "dependsOn": [
        "[concat('Microsoft.Resources/deployments/','BuildIpConfigurations')]"
      ],
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[concat(variables('interfacesUri'), '/networkInterfaces.json', parameters('SasToken'))]",
          "contentVersion": "2017.09.01.0"
        },
        "parameters": {
          "name": {
            "value": "[variables('interfaceName')]"
          },
          "ipConfigurations": {
            "value": "[createarray(reference('BuildIpConfigurations').outputs.ipConfigurations.value)]"
          },
          "tags": {
            "value": "[json('{\"TagName\": \"TagValue\"}')]"
          }
        }
      }
    },
    {
      "name": "DeployNetworkInterface",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2016-09-01",
      "dependsOn": [
        "[concat('Microsoft.Resources/deployments/','CreateNetworkInterface')]"
      ],
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[concat(variables('computeUri'), '/networkInterfaces.json', parameters('SasToken'))]",
          "contentVersion": "2017.09.01.0"
        },
        "parameters": {
          "id": {
            "value": "[resourceId(resourceGroup().name,'Microsoft.Network/networkInterfaces',variables('interfaceName'))]"
          },
          "primary": {
            "value": true
          }
        }
      }
    },
    {
      "name": "BuildNetworkProfile",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2016-09-01",
      "dependsOn": [
        "[concat('Microsoft.Resources/deployments/','DeployNetworkInterface')]"
      ],
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[concat(variables('computeUri'), '/networkProfile.json', parameters('SasToken'))]",
          "contentVersion": "2017.09.01.0"
        },
        "parameters": {
          "networkInterfaces": {
            "value": "[createarray(reference('DeployNetworkInterface').outputs.networkInterfaces.value)]"
          }
        }
      }
    },
    {
      "name": "BuildStorageProfile",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2016-09-01",
      "dependsOn": [
        "[concat('Microsoft.Resources/deployments/','BuildImageReference')]",
        "[concat('Microsoft.Resources/deployments/','BuildOsDisk')]",
        "[concat('Microsoft.Resources/deployments/','BuildDataDisk')]"
      ],
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[concat(variables('computeUri'), '/storageProfile.json', parameters('SasToken'))]",
          "contentVersion": "2017.09.01.0"
        },
        "parameters": {
          "imageReference": {
            "value": "[reference('BuildImageReference').outputs.imageReference.value]"
          },
          "osDisk": {
            "value": "[reference('BuildOsDisk').outputs.osDisk.value]"
          },
          "dataDisks": {
            "value": "[createarray(reference('BuildDataDisk').outputs.dataDisks.value)]"
          }
        }
      }
    },
    {
      "name": "CreateDataDisk",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2016-09-01",
      "dependsOn": [ ],
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[concat(variables('disksUri'), '/disks.json', parameters('SasToken'))]",
          "contentVersion": "2017.09.01.0"
        },
        "parameters": {
          "name": {
            "value": "[concat(variables('computerName'),'-datadisk-1.vhd')]"
          },
          "createOption": {
            "value": "Empty"
          },
          "diskSizeGB": {
            "value": 512
          },
          "sku": {
            "value": "Premium_LRS"
          },
          "tags": {
            "value": "[json('{\"TagName\": \"TagValue\"}')]"
          }
        }
      }
    },
    {
      "name": "BuildDataDisk",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2016-09-01",
      "dependsOn": [
        "[concat('Microsoft.Resources/deployments/','CreateDataDisk')]"
      ],
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[concat(variables('computeUri'), '/dataDisks.json', parameters('SasToken'))]",
          "contentVersion": "2017.09.01.0"
        },
        "parameters": {
          "name": {
            "value": "[concat(variables('computerName'),'-datadisk-1.vhd')]"
          },
          "diskSizeGB": {
            "value": 512
          },
          "caching": {
            "value": "ReadOnly"
          },
          "lun": {
            "value": 1
          },
          "ManagedDiskId": {
            "value": "[resourceId(resourceGroup().name,'Microsoft.Compute/disks',concat(variables('computerName'),'-datadisk-1.vhd'))]"
          },
          "createOption": {
            "value": "Attach"
          }
        }
      }
    },
    {
      "name": "BuildOsDisk",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2016-09-01",
      "dependsOn": [ ],
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[concat(variables('computeUri'), '/osDisk.json', parameters('SasToken'))]",
          "contentVersion": "2017.09.01.0"
        },
        "parameters": {
          "name": {
            "value": "[concat(variables('computerName'),'.vhd')]"
          },
          "caching": {
            "value": "ReadOnly"
          },
          "createOption": {
            "value": "FromImage"
          }
        }
      }
    },
    {
      "name": "BuildImageReference",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2016-09-01",
      "dependsOn": [ ],
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[concat(variables('computeUri'), '/imageReference.json', parameters('SasToken'))]",
          "contentVersion": "2017.09.01.0"
        },
        "parameters": {
          "publisher": {
            "value": "[concat(variables('OS')[parameters('OperatingSystem')].imagePublisher)]"
          },
          "offer": {
            "value": "[concat(variables('OS')[parameters('OperatingSystem')].imageOffer)]"
          },
          "sku": {
            "value": "[concat(variables('OS')[parameters('OperatingSystem')].imageSKU)]"
          },
          "version": {
            "value": "[concat(variables('OS')[parameters('OperatingSystem')].version)]"
          }
        }
      }
    },
    {
      "name": "DefineWindowsConfiguration",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2016-09-01",
      "dependsOn": [ ],
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[concat(variables('computeUri'), '/windowsConfiguration.json', parameters('SasToken'))]",
          "contentVersion": "2017.09.01.0"
        },
        "parameters": {
          "provisionVMAgent": {
            "value": true
          },
          "winRM": {
            "value": { }
          },
          "additionalUnattendContent": {
            "value": { }
          },
          "enableAutomaticUpdates": {
            "value": true
          },
          "timeZone": {
            "value": "[variables('timeZone')]"
          }
        }
      }
    },
    {
      "name": "BuildOsProfile",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2016-09-01",
      "dependsOn": [
        "[concat('Microsoft.Resources/deployments/','DefineWindowsConfiguration')]"
      ],
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[concat(variables('computeUri'), '/osProfile.json', parameters('SasToken'))]",
          "contentVersion": "2017.09.01.0"
        },
        "parameters": {
          "computerName": {
            "value": "[variables('computerName')]"
          },
          "adminUsername": {
            "value": "[parameters('AdminUsername')]"
          },
          "adminPassword": {
            "value": "[parameters('AdminPassword')]"
          },
          "customData": {
            "value": ""
          },
          "windowsConfiguration": {
            "value": "[reference('DefineWindowsConfiguration').outputs.windowsConfiguration.value]"
          },
          "linuxConfiguration": {
            "value": { }
          },
          "secrets": {
            "value": { }
          }
        }
      }
    },
    {
      "name": "BuildHardwareProfile",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2016-09-01",
      "dependsOn": [ ],
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[concat(variables('computeUri'), '/hardwareProfile.json', parameters('SasToken'))]",
          "contentVersion": "2017.09.01.0"
        },
        "parameters": {
          "vmSize": {
            "value": "[variables('vmSize')]"
          }
        }
      }
    },
    {
      "name": "CreateDiagnosticsStorageAccount",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2016-09-01",
      "dependsOn": [ ],
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[concat(variables('StorageUri'), '/storageAccounts.json', parameters('SasToken'))]",
          "contentVersion": "2017.09.01.0"
        },
        "parameters": {
          "name": {
            "value": "[variables('DiagnosticsStorageAccount')]"
          },
          "sku": {
            "value": "Standard_LRS"
          },
          "kind": {
            "value": "Storage"
          }
        }
      }
    },
    {
      "name": "BuildBootDiagnostics",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2016-09-01",
      "dependsOn": [
        "[concat('Microsoft.Resources/deployments/','CreateDiagnosticsStorageAccount')]"
      ],
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[concat(variables('computeUri'), '/bootDiagnostics.json', parameters('SasToken'))]",
          "contentVersion": "2017.09.01.0"
        },
        "parameters": {
          "enabled": {
            "value": "true"
          },
          "storageUri": {
            "value": "[reference(concat('Microsoft.Storage/storageAccounts/', variables('DiagnosticsStorageAccount')), '2016-01-01').primaryEndpoints.blob]"
          }
        }
      }
    },
    {
      "name": "BuildDiagnosticsProfile",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2016-09-01",
      "dependsOn": [
        "[concat('Microsoft.Resources/deployments/','BuildBootDiagnostics')]"
      ],
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[concat(variables('computeUri'), '/diagnosticsProfile.json', parameters('SasToken'))]",
          "contentVersion": "2017.09.01.0"
        },
        "parameters": {
          "bootDiagnostics": {
            "value": "[reference('BuildBootDiagnostics').outputs.bootDiagnostics.value]"
          }
        }
      }
    }
  ],
  "outputs": {
    "virtualMachines": {
      "type": "object",
      "value": "[reference('BuildVirtualMachine').outputs.virtualMachine.value]"
    }
  }
}
```
