# Build-Azure-VM-with-Packer
1.	Create Resource group
```
$location = "East US 2" 
$groupName = "testGroup1"
az group create --name $groupName --location $location
```
2.	Create strorage account
3.	Create service principle
4.	Build VHD with packer template
5.	Create vm from VHD
