---
author: KarlErickson
ms.author: haiche
ms.date: 11/30/2022
---

5. Use the [az vm start](/cli/azure/vm#az-vm-start) command to start `adminVM`.

   ```azurecli
   az vm start --resource-group abc1110rg --name adminVM
   ```

   Use the following commands to get and show the private IP addresses, which you use in later sections.

   ```azurecli
   export ADMINVM_NIC_ID=$(az vm show \
       --resource-group abc1110rg \
       --name adminVM \
       --query networkProfile.networkInterfaces[0].id \
       --output tsv)
   export ADMINVM_IP=$(az network nic show \
       --ids ${ADMINVM_NIC_ID} \
       --query ipConfigurations[0].privateIPAddress \
       --output tsv)
   export MSPVM1_NIC_ID=$(az vm show \
       --resource-group abc1110rg \
       --name mspVM1 \
       --query networkProfile.networkInterfaces[0].id \
       --output tsv)
   export MSPVM1_IP=$(az network nic show \
       --ids ${MSPVM1_NIC_ID} \
       --query ipConfigurations[0].privateIPAddress \
       --output tsv)
   export MSPVM2_NIC_ID=$(az vm show \
       --resource-group abc1110rg \
       --name mspVM2 \
       --query networkProfile.networkInterfaces[0].id \
       --output tsv)
   export MSPVM2_IP=$(az network nic show \
       --ids ${MSPVM2_NIC_ID} \
       --query ipConfigurations[0].privateIPAddress \
       --output tsv)
   echo "Private IP of adminVM: ${ADMINVM_IP}"
   echo "Private IP of mspVM1: ${MSPVM1_IP}"
   echo "Private IP of mspVM2: ${MSPVM2_IP}"
   ```
