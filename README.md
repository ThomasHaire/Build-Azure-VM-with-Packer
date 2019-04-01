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
