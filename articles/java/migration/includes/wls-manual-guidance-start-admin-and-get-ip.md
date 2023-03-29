---
author: KarlErickson
ms.author: haiche
ms.date: 11/30/2022
---

1. Start **adminVM**.

   Use the [az vm start](/cli/azure/vm#az-vm-start) command to start the VM.

   ```azurecli-interactive
   az vm start --resource-group abc1110rg --name adminVM
   ```

   Use the following commands to get and show the private IP addresses, which you'll use in later sections.

   ```azurecli-interactive
   ADMINVM_NIC_ID=$(az vm show --resource-group abc1110rg --name adminVM --query networkProfile.networkInterfaces[0].id --output tsv)
   ADMINVM_IP=$(az network nic show --ids ${ADMINVM_NIC_ID} --query ipConfigurations[0].privateIPAddress --output tsv)
   MSPVM1_NIC_ID=$(az vm show --resource-group abc1110rg --name mspVM1 --query networkProfile.networkInterfaces[0].id --output tsv)
   MSPVM1_IP=$(az network nic show --ids ${MSPVM1_NIC_ID} --query ipConfigurations[0].privateIPAddress --output tsv)
   MSPVM2_NIC_ID=$(az vm show --resource-group abc1110rg --name mspVM2 --query networkProfile.networkInterfaces[0].id --output tsv)
   MSPVM2_IP=$(az network nic show --ids ${MSPVM2_NIC_ID} --query ipConfigurations[0].privateIPAddress --output tsv)
   echo "Private IP of adminVM: ${ADMINVM_IP}"
   echo "Private IP of mspVM1: ${MSPVM1_IP}"
   echo "Private IP of mspVM2: ${MSPVM2_IP}"
   ```
