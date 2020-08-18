# kubernetesplayground

Simple Kubernetes cluster playground set up using Azure VMs, CLI and bash. Note! You need to have init-master.sh and init-node.sh in the same folder from where you run the commands, together with you public ssh key.

For more information, check my [blog post](https://building4.cloud/set-up-kubernetes-cluster-playground-with-azure-virtual-machines-and-cli.html).

I have already setup vnet with two subnets (management-subnet and kubernetes-subnet). I'm using Jumpbox for connecting the cluster resources. Jumpbox is already provisioned to management-subnet. 

Following script can be used to provision the VMs.

```bash
RESOURCEGROUP=rg-kubernetes-test
LOCATION=westeurope
ADMINUSER=azureuser
VM_SIZE=Standard_D2_v3
VNET_RG=rg-networking
VNET=vnet-test 
SUBNETNAME=kubernetes-subnet
SUBNETID=$(az network vnet subnet show --resource-group $VNET_RG --name $SUBNETNAME --vnet-name $VNET --query="id" -o tsv)

az group create --name $RESOURCEGROUP --location $LOCATION

az vm create --name master1 --resource-group $RESOURCEGROUP --location $LOCATION --admin-username $ADMINUSER --size $VM_SIZE --image UbuntuLTS --subnet $SUBNETID --public-ip-address "" --nsg "" --ssh-key-values ./id_rsa.pub --custom-data ./init-master.sh --no-wait
az vm create --name node1 --resource-group $RESOURCEGROUP --location $LOCATION --admin-username $ADMINUSER --size $VM_SIZE --image UbuntuLTS --subnet $SUBNETID --public-ip-address "" --nsg "" --ssh-key-values ./id_rsa.pub --custom-data ./init-node.sh --no-wait
az vm create --name node2 --resource-group $RESOURCEGROUP --location $LOCATION --admin-username $ADMINUSER --size $VM_SIZE --image UbuntuLTS --subnet $SUBNETID --public-ip-address "" --nsg "" --ssh-key-values ./id_rsa.pub --custom-data ./init-node.sh --no-wait
```

Nodes will need to be joined to the cluster. You can create a new token by using ´´´kubeadm join´´´.

```bash
ssh azureuser@ip-address-of-master
kubeadm token create --print-join-command
```

Then run the `kubeadm join` command which is outputted on both nodes. Have fun!

And finally remember to remove the resource group to avoid unnecessary costs.

```bash
 az group delete --name rg-kubernetes-test --yes --no-wait
```
