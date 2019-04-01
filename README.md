# Build-Azure-VM-with-Packer
1.	Create Resource group
```
$location = "East US 2" 
$groupName = "testGroup1"
az group create --name $groupName --location $location
```
2.	Create strorage account
```
$location = "East US 2" 
$groupName = "testGroup1"
az storage account create --name teststorage4119 --resource-group $groupName --location $location --sku Standard_LRS --kind Storage
```
3.	Create service principle
```
$subscriptionId=$(az account show --output tsv --query id)
$name="http://packer-${subscriptionId}-sp"
az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/$subscriptionId" --name "$name" --output json
```
4.	Build VHD with Packer template
```
{
    "builders": [
        {
            "type": "azure-arm",
            "client_id": "appId",
            "client_secret": "password",
            "tenant_id": "tenant",
            "subscription_id": "subscriptionId",
            
            "managed_image_resource_group_name": "resourceGroupName",
            "managed_image_name": "firstImage",
            
            "os_type": "Windows",
            "image_publisher": "MicrosoftWindowsServer",
            "image_offer": "WindowsServer",
            "image_sku": "2016-Datacenter",
            
            "communicator": "winrm",
            "winrm_use_ssl": true,
            "winrm_insecure": true,
            "winrm_timeout": "3m",
            "winrm_username": "packer",
           
            "azure_tags": {
                "dept": "test",
                "task": "test deployment"
            },
            
            "location": "East US",
            "vm_size": "Standard_DS2_v2"
        }
    ],
    "provisioners": [
        {
            "type": "powershell",
            "inline": [
                "Add-WindowsFeature Web-Server",
                "& $env:SystemRoot\\System32\\Sysprep\\Sysprep.exe /oobe /generalize /quiet /quit",
                "while($true) { $imageState = Get-ItemProperty HKLM:\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Setup\\State | Select ImageState; if($imageState.ImageState -ne 'IMAGE_STATE_GENERALIZE_RESEAL_TO_OOBE') { Write-Output $imageState.ImageState; Start-Sleep -s 10  } else { break } }"
            ]
        }
    ]
}
```
```
packer build template.json
```
5.	Create VM from VHD
```
New-AzVm `
    -ResourceGroupName $rgName `
    -Name "myVM" `
    -Location $location `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -PublicIpAddressName "myPublicIpAddress" `
    -OpenPorts 80 `
    -Image "myPackerImage"
```
