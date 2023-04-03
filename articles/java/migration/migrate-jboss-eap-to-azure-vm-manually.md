---
title: "Tutorial: Install Red Hat JBoss EAP on Azure Virtual Machines manually"
description: Provides step-by-step guidance to install Red Hat JBoss EAP on Azure VMs and form a cluster, expose it with Azure Application Gateway, and connect with Azure Database for PostgreSQL.
author: KarlErickson
ms.author: zhengchang
ms.topic: how-to
ms.date: 03/20/2023
recommendations: false
ms.custom: devx-track-java, devx-track-javaee, devx-track-javaee-jbosseap, devx-track-javaee-jbosseap-vm, migration-java, devx-track-azurecli
---

# Tutorial: Install Red Hat JBoss EAP on Azure Virtual Machines manually

This tutorial shows the steps to install Red Hat JBoss EAP and configure a cluster in domain mode on Azure Virtual Machines (VMs), on Red Hat Enterprise Linux(RHEL).

In this tutorial, you learn how to:

> [!div class="checklist"]
>
> - Create a custom virtual network and create the VMs within the network.
> - Install the desired JDK and Red Hat JBoss EAP on the VMs by using the command-line manually.
> - Configure a Red Hat JBoss EAP cluster in domain mode using the command-line interface(CLI).
> - Configure a PostgreSQL datasource connection in the cluster.
> - Deploy and run a sample Java EE application in the cluster.
> - Expose the application to the public internet via Azure Application Gateway.
> - Validate the successful configuration.

If you prefer a fully automated solution that does all of these steps on your behalf on GNU/Linux VMs, directly from the Azure portal, see [Quickstart: Deploy JBoss EAP Server on an Azure virtual machine using the Azure portal](../ee/jboss-eap-single-server-azure-vm.md).

## Prerequisites

- [!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]
- [Install Azure CLI version 2.46.0 or higher](/cli/azure/install-azure-cli) to run Azure CLI commands.
  - When you're prompted, install Azure CLI extensions on first use. For more information about extensions, see [Use extensions with Azure CLI](/cli/azure/azure-cli-extensions-overview).
  - Run [az version](/cli/azure/reference-index?#az-version) to find the version and dependent libraries that are installed. To upgrade to the latest version, run [az upgrade](/cli/azure/reference-index?#az-upgrade).
- Ensure you have the necessary Red Hat licenses. You need to have a Red Hat Account with Red Hat Subscription Management (RHSM) entitlement for Red Hat JBoss EAP. This entitlement will let the Azure portal install the Red Hat tested and certified JBoss EAP version.
  > [!NOTE]
  > If you don't have an EAP entitlement, you can sign up for a free developer subscription through the [Red Hat Developer Subscription for Individuals](https://developers.redhat.com/register). Write down the account details, which will be used as the *RHSM username* and *RHSM password* in the next section.
- After you're registered, you can find the necessary credentials (*Pool IDs*) by following steps below. The *Pool IDs* will also be used as the *RHSM Pool ID with EAP entitlement* later.
  - Sign in to your [Red Hat account](https://sso.redhat.com).
  - The first time you sign in, you'll be asked to complete your profile. Make sure you select **Personal** for the **Account Type**, as shown in the following screenshot.

    :::image type="content" source="media/migrate-jboss-eap-to-vm-manually/update-account-type-as-personal.png" alt-text="Screenshot of selecting 'Personal' for the 'Account Type'." lightbox="media/migrate-jboss-eap-to-vm-manually/update-account-type-as-personal.png":::

  - In the tab where you're signed in, open [Red Hat Developer Subscription for Individuals](https://aka.ms/red-hat-individual-dev-sub). This link takes you to all of the subscriptions in your account for the appropriate SKU.
  - Select the first subscription from the **All purchased Subscriptions** table.
  - Copy and write down the value following **Master Pools** from **Pool IDs**.

- A Java JDK, Version 11. In this guide we recommend [Red Hat Build of OpenJDK](https://developers.redhat.com/products/openjdk/download). Ensure that your `JAVA_HOME` environment variable is set correctly in the shells in which you run the commands.
- [Git](https://git-scm.com/downloads); use `git --version` to test whether `git` works. This tutorial was tested with version 2.25.1.
- [Maven](https://maven.apache.org/download.cgi); use `mvn -version` to test whether `mvn` works. This tutorial was tested with version 3.6.3.

## Prepare the environment

In this section, you'll set up the infrastructure within which you'll install the JDK, Red Hat JBoss EAP, and the PostgreSQL JDBC driver.

### Assumptions

This tutorial will configure a Red Hat JBoss EAP cluster in domain mode with an administration server and two managed servers on a total of three VMs. To configure the cluster, you need to create the following three Azure VMs:

- An admin VM (VM name `adminVM`) runs as the domain controller.
- Two managed VMs (VM names `mspVM1` and `mspVM2`) run as host controller.

### Sign in to Azure

If you haven't already, sign in to your Azure subscription by using the [az login](/cli/azure/reference-index) command and follow the on-screen directions.

```azurecli
az login
```

> [!NOTE]
> If you have multiple Azure tenants associated with your Azure credentials, you must specify which tenant you want to sign in to. You can do this with the `--tenant` option. For example, `az login --tenant contoso.onmicrosoft.com`.

### Create a resource group

Create a resource group with [az group create](/cli/azure/group#az-group-create). Resource group names must be globally unique within a subscription. For this reason, consider prepending some unique identifier to any names you create that must be unique. A useful technique is to use your initials followed by today's date in `mmdd` format. This example creates a resource group named `abc1110rg` in the `eastus` location:

```azurecli
az group create \
    --name abc1110rg \
    --location eastus
```

### Create a virtual network

The resources comprising your Red Hat JBoss EAP cluster must communicate with each other, and the public internet, using a virtual network. For a complete guide to planning your virtual network, see the Cloud Adoption Framework for Azure guide [Plan virtual networks](/azure/virtual-network/virtual-network-vnet-plan-design-arm). For more information, see [Azure Virtual Network frequently asked questions](/azure/virtual-network/virtual-networks-faq).

The example in this section creates a virtual network with address space `192.168.0.0/16` and creates a subnet used for VMs.

First, create a virtual network by using [az network vnet create](/cli/azure/network/vnet#az-network-vnet-create). The following example creates a network named `myVNet`:

```azurecli
az network vnet create \
    --resource-group abc1110rg \
    --name myVNet \
    --address-prefixes 192.168.0.0/24
```

Create a subnet for the Red Hat JBoss EAP cluster by using [az network vnet subnet create](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-create). The following example creates a subnet named `mySubnet`:

```azurecli
az network vnet subnet create \
    --resource-group abc1110rg \
    --name mySubnet \
    --vnet-name myVNet \
    --address-prefixes 192.168.0.0/25
```

Create a subnet for Application Gateway by using [az network vnet subnet create](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-create). The following example creates a subnet named `jbossVMGatewaySubnet`:

```azurecli
az network vnet subnet create \
    --resource-group abc1110rg \
    --name jbossVMGatewaySubnet \
    --vnet-name myVNet \
    --address-prefixes 192.168.0.128/25
```

### Create a Network Security Group(NSG) and assign subnets to it

Before we create VMs with public IPs, to secure the virtual network and subnets created above, let's create a network security group.

Create a network security group by using [az network nsg create](/cli/azure/network/nsg?view=azure-cli-latest#az-network-nsg-create). The following example creates a network security group named `mynsg`:

```azurecli
az network nsg create \
    --resource-group abc1110rg \
    --name mynsg
```

Create network security group rules by using [az network nsg rule create](/cli/azure/network/nsg/rule?view=azure-cli-latest#az-network-nsg-rule-create). The following example creates network security group rules named `ALLOW_APPGW` and `ALLOW_HTTP_ACCESS`. These rules allow App Gateway to accept inbound traffic on the HTTP ports used by Red Hat JBoss EAP:

```azurecli
az network nsg rule create --resource-group abc1110rg --nsg-name mynsg --name ALLOW_APPGW --protocol Tcp --destination-port-ranges 65200-65535 --source-address-prefix GatewayManager --destination-address-prefix '*' --access Allow --priority 500 --direction Inbound

az network nsg rule create --resource-group abc1110rg --nsg-name mynsg --name ALLOW_HTTP_ACCESS --protocol Tcp --destination-port-ranges 22 80 443 9990 8080 --source-address-prefix Internet --destination-address-prefix '*' --access Allow --priority 510 --direction Inbound
```

Associate the subnets created above to this network security group by using [az network vnet subnet update](/cli/azure/network/vnet/subnet?view=azure-cli-latest#az-network-vnet-subnet-update).

```azurecli
az network vnet subnet update --vnet-name myVNet --name mySubnet --network-security-group mynsg --resource-group abc1110rg

az network vnet subnet update --vnet-name myVNet --name jbossVMGatewaySubnet --network-security-group mynsg --resource-group abc1110rg
```

### Create a Red Hat Enterprise Linux machine for admin

#### Create the admin VM

The Marketplace image that you use to create the VMs is `RedHat:RHEL:8_6:latest`. For other images, please see [Red Hat Enterprise Linux (RHEL) images available in Azure](/azure/virtual-machines/workloads/redhat/redhat-imagelist).

> [!NOTE]
> You can query all the available Red Hat Enterprise Linux images provided by Oracle with [az vm image list](/cli/azure/vm/image#az-vm-image-list) `az vm image list --offer RHEL --publisher RedHat --output table --all`. For more information, see [Oracle VM images and their deployment on Microsoft Azure](/azure/virtual-machines/workloads/oracle/oracle-vm-solutions).
>
> If you use a different image, you may need to install extra libraries to enable the infrastructure used in this guide.

You'll create a basic VM, install all required tools on it, take a snapshot of it, and then create replicas based on the snapshot.

Create a VM using [az vm create](/cli/azure/vm). You'll run the Administration Server on this VM.

The following example creates a Red Hat Enterprise Linux VM using user name and password pair for the authentication. If desired, you can use SSL authentication instead.

```azurecli
az vm create \
    --resource-group abc1110rg \
    --name adminVM \
    --image RedHat:RHEL:8_6:latest \
    --size Standard_DS1_v2  \
    --admin-username azureuser \
    --admin-password Secret123456 \
    --public-ip-sku Standard \
    --nsg mynsg \
    --vnet-name myVnet \
    --subnet mySubnet
```

#### Install OpenJDK 11 and Red Hat JBoss EAP 7.4

1. Use the following command to get the public IP of `adminVM`
    
    ```azurecli
    ADMIN_VM_PUBLIC_IP=$(az vm show --resource-group abc1110rg --name adminVM --show-details --query publicIps | tr -d '"')
    ```

2. Open a terminal and SSH to the `adminVM` by running the following command:
    
    ```bash
    ssh azureuser@$ADMIN_VM_PUBLIC_IP
    ```

Provide `Secret123456` as password.

3. Configure firewall for ports by running:
    
    ```bash
    sudo firewall-cmd --zone=public --add-port={9999/tcp,8443/tcp,8009/tcp,8080/tcp,9990/tcp,9993/tcp,45700/tcp,7600/tcp} --permanent
    sudo firewall-cmd --reload
    sudo iptables-save
    ```
    
    You should see the word `success` after the first two commands. You should see output similar to the following after the third command.
    
    ```bash
    # Generated by iptables-save v1.8.4 on Wed Mar 29 22:39:23 2023
    *filter
    :INPUT ACCEPT [20:3546]
    :FORWARD ACCEPT [0:0]
    :OUTPUT ACCEPT [24:5446]
    COMMIT
    # Completed on Wed Mar 29 22:39:23 2023
    # Generated by iptables-save v1.8.4 on Wed Mar 29 22:39:23 2023
    *security
    :INPUT ACCEPT [19:3506]
    :FORWARD ACCEPT [0:0]
    :OUTPUT ACCEPT [5:492]
    -A OUTPUT -d 168.63.129.16/32 -p tcp -m tcp --dport 53 -j ACCEPT
    -A OUTPUT -d 168.63.129.16/32 -p tcp -m tcp --dport 53 -j ACCEPT
    -A OUTPUT -d 168.63.129.16/32 -p tcp -m owner --uid-owner 0 -j ACCEPT
    -A OUTPUT -d 168.63.129.16/32 -p tcp -m conntrack --ctstate INVALID,NEW -j DROP
    COMMIT
    # Completed on Wed Mar 29 22:39:23 2023
    # Generated by iptables-save v1.8.4 on Wed Mar 29 22:39:23 2023
    *raw
    :PREROUTING ACCEPT [20:3546]
    :OUTPUT ACCEPT [24:5446]
    COMMIT
    # Completed on Wed Mar 29 22:39:23 2023
    # Generated by iptables-save v1.8.4 on Wed Mar 29 22:39:23 2023
    *mangle
    :PREROUTING ACCEPT [20:3546]
    :INPUT ACCEPT [20:3546]
    :FORWARD ACCEPT [0:0]
    :OUTPUT ACCEPT [24:5446]
    :POSTROUTING ACCEPT [24:5446]
    COMMIT
    # Completed on Wed Mar 29 22:39:23 2023
    # Generated by iptables-save v1.8.4 on Wed Mar 29 22:39:23 2023
    *nat
    :PREROUTING ACCEPT [1:40]
    :INPUT ACCEPT [0:0]
    :POSTROUTING ACCEPT [4:240]
    :OUTPUT ACCEPT [4:240]
    COMMIT
    # Completed on Wed Mar 29 22:39:23 2023
    ```

4. Register the admin host to your Red Hat Subscription Management(RHSM) account
    
    ```bash
    RHSM_USER=<your rhsm username>
    RHSM_PASSWORD="<your rhsm password>"
    EAP_POOL=<your rhsm pool id>
    
    sudo subscription-manager register --username ${RHSM_USER} --password ${RHSM_PASSWORD} --force
    ```
    
    You should see output similar to the following.
    
    ```bash
    Registering to: subscription.rhsm.redhat.com:443/subscription
    The system has been registered with ID: redacted
    The registered system name is: adminVM
    ```

5. Attach the admin host to Red Hat JBoss EAP pool
    
    ```bash
    sudo subscription-manager attach --pool=${EAP_POOL}
    ```

6. Install OpenJDK 11
    
    ```bash
    sudo yum install java-11-openjdk -y
    ```
    
    You should see many lines of output, ending with `Complete!`.

7. Install Red Hat JBoss EAP 7.4

    ```bash
    sudo subscription-manager repos --enable=jb-eap-7.4-for-rhel-8-x86_64-rpms
    sudo yum update -y --disablerepo='*' --enablerepo='*microsoft*'
    sudo yum groupinstall -y jboss-eap7
    ```

    For the second and third commands, you should see many lines of output, ending with `Complete!`.

8. Set permission and TCP configurations

    ```bash
    sudo sed -i 's/PermitRootLogin yes/PermitRootLogin no/g' /etc/ssh/sshd_config
    echo 'AllowTcpForwarding no' | sudo tee -a /etc/ssh/sshd_config
    sudo systemctl restart sshd
    ```

9. Configure the environment variables

    ```bash
    echo 'export EAP_RPM_CONF_DOMAIN="/etc/opt/rh/eap7/wildfly/eap7-domain.conf"' >> ~/.bash_profile
    echo 'export EAP_HOME="/opt/rh/eap7/root/usr/share"' >> ~/.bash_profile
    source ~/.bash_profile
    sudo touch /etc/profile.d/eap_env.sh
    echo 'export EAP_HOME="/opt/rh/eap7/root/usr/share"' | sudo tee -a /etc/profile.d/eap_env.sh
    ```
    
Exit from the ssh connection by typing `exit`.

### Create machines for managed servers

You've installed OpenJDK 11, and Red Hat JBoss EAP 7.4 on `adminVM`, which will run as the domain controller server. You still need to prepare machines to run the two host controller servers. Next, you'll create a snapshot of `adminVM` and prepare machines for two managed severs, `mspVM1` and `mspVM2`.

This section introduces an approach to prepare machines with the snapshot of `adminVM`. Return to your terminal that has Azure CLI signed in, then follow these steps:

1. Use the following command to stop `adminVM`:
    
    ```azurecli
    az vm stop --resource-group abc1110rg --name adminVM
    ```

2. Use [az snapshot create](/cli/azure/snapshot#az-snapshot-create) to take a snapshot of the `adminVM` OS disk.

    ```azurecli
    ADMIN_OS_DISK_ID=$(az vm show \
        --resource-group abc1110rg \
        --name adminVM \
        --query storageProfile.osDisk.managedDisk.id \
        --output tsv)
    az snapshot create \
        --resource-group abc1110rg \
        --name myAdminOSDiskSnapshot \
        --source ${ADMIN_OS_DISK_ID}
    ```

3. Use the following steps to create `mspVM1`.

    First, create a managed disk for `mspVM1` with [az disk create](/cli/azure/disk#az-disk-create):
    
    ```azurecli
    #Get the snapshot ID
    SNAPSHOT_ID=$(az snapshot show \
        --name myAdminOSDiskSnapshot \
        --resource-group abc1110rg \
        --query '[id]' \
        --output tsv)
    
    #Create a new Managed Disks using the snapshot Id
    #Note that managed disk will be created in the same location as the snapshot
    az disk create \
        --resource-group abc1110rg \
        --name mspVM1_OsDisk_1 \
        --source ${SNAPSHOT_ID}
    ```
    
    Next, create VM `mspVM1`, attaching OS disk `mspVM1_OsDisk_1`:
    
    ```azurecli
    #Get the resource Id of the managed disk
    MSPVM1_DISK_ID=$(az disk show \
        --name mspVM1_OsDisk_1 \
        --resource-group abc1110rg \
        --query '[id]' \
        --output tsv)
    
    #Create VM by attaching existing managed disks as OS
    az vm create \
        --resource-group abc1110rg \
        --name mspVM1 \
        --attach-os-disk ${MSPVM1_DISK_ID} \
        --os-type linux \
        --public-ip-sku Standard \
        --nsg mynsg \
        --vnet-name myVnet \
        --subnet mySubnet
    ```

    You've created `mspVM1` with OpenJDK 11 and Red Hat JBoss EAP 7.4 installed. Because the VM was created from a snapshot of the `adminVM` OS disk, the two VMs have the same hostname. Use [az vm run-command invoke](/cli/azure/vm/run-command#az-vm-run-command-invoke) to change the hostname to the value `mspVM1`:

    ```azurecli
    az vm run-command invoke \
        --resource-group abc1110rg \
        --name mspVM1 \
        --command-id RunShellScript \
        --scripts "sudo hostnamectl set-hostname mspVM1"
    ```

    When the command completes successfully, you'll see output similar to this:
    ```json
    {
        "value": [
            {
            "code": "ProvisioningState/succeeded",
            "displayStatus": "Provisioning succeeded",
            "level": "Info",
            "message": "Enable succeeded: \n[stdout]\n\n[stderr]\n",
            "time": null
            }
        ]
    }
    ```

4. Use the same steps to create `mspVM2`.

    The steps to create `mspVM2` are the same as creating `mspVM1`.
    ```azurecli
    #Create a new Managed Disks for mspVM2
    az disk create --resource-group abc1110rg --name mspVM2_OsDisk_1 --source ${SNAPSHOT_ID}
    
    #Get the resource Id of the managed disk
    MSPVM2_DISK_ID=$(az disk show \
    --name mspVM2_OsDisk_1 \
    --resource-group abc1110rg \
    --query '[id]' \
    --output tsv)
    
    #Create VM by attaching existing managed disks as OS
    az vm create \
        --resource-group abc1110rg \
        --name mspVM2 \
        --attach-os-disk ${MSPVM2_DISK_ID} \
        --os-type linux \
        --public-ip-sku Standard \
        --nsg mynsg \
        --vnet-name myVnet \
        --subnet mySubnet
    
    #Set hostname
    az vm run-command invoke \
        --resource-group abc1110rg \
        --name mspVM2 \
        --command-id RunShellScript \
        --scripts "sudo hostnamectl set-hostname mspVM2"
    ```
[!INCLUDE [start-admin-get-ips](includes/wls-manual-guidance-start-admin-and-get-ip.md)]

Now, all three machines are ready. Next, you'll configure a Red Hat JBoss EAP cluster in managed domain mode.

### Configure managed domain and cluster

Configure the cluster with session replication enabled. For more information, see [Session Replication](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.4/html/development_guide/clustering_in_web_applications#session_replication). 

To enable session replication we will use Red Hat JBoss EAP High Availability for the cluster. Microsoft Azure does not support JGroups discovery protocols that are based on UDP multicast. Although you may use other JGroups discovery protocols (such as a static configuration (`TCPPING`), a shared database (`JDBC_PING`), shared file system-based ping (`FILE_PING`), or `TCPGOSSIP`), we strongly recommend that you use the shared file discovery protocol specifically developed for Azure: `AZURE_PING`. For more information, see [Using JBoss EAP High Availability in Microsoft Azure](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.4/html/using_jboss_eap_in_microsoft_azure/using_jboss_eap_high_availability_in_microsoft_azure#doc-wrapper) 

#### Create Azure storage account and Blob container for AZURE_PING

Use the following command to create a storage account and Blob container:

```azurecli
# Define your storage account name
STORAGE_ACCOUNT_NAME=azurepingstgabc1110rg
# Define your Blob container name
CONTAINER_NAME=azurepingcontainerabc1110rg

# Create storage account
az storage account create \
  --name ${STORAGE_ACCOUNT_NAME} \
  --resource-group abc1110rg \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2 \
  --access-tier Hot
```

Then, retrieve the storage account key for later use by running the following command:

```bash
STORAGE_ACCESS_KEY=$(az storage account keys list \
  --account-name ${STORAGE_ACCOUNT_NAME} \
  --query "[0].value" --output tsv)
```

```bash
# Create blob container
az storage container create \
  --name ${CONTAINER_NAME} \
  --account-name ${STORAGE_ACCOUNT_NAME} \
  --account-key ${STORAGE_ACCESS_KEY}
```

You should see the following output.

```bash
{
  "created": true
}
```


#### Configure domain controller(admin node)

This tutorial uses the Red Hat JBoss EAP management CLI commands to configure the domain controller. For more information, see [Management CLI Guide](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.4/html-single/management_cli_guide/index).

The following steps set up the domain controller configuration on `adminVM`. Use ssh to connect to the `adminVM` as the `azureuser` user. Recall that the public IP address of the adminVM was captured above into the `ADMIN_VM_PUBLIC_IP` environment variable.

First, configure the HA profile and JGroups using **AZURE_PING** protocol.

```bash
HOST_VM_IP=$(hostname -I)
STORAGE_ACCOUNT_NAME=azurepingstgabc1110rg
CONTAINER_NAME=azurepingcontainerabc1110rg
STORAGE_ACCESS_KEY=<the value from before you connected with ssh>


# Configure the HA profile and JGroups using AZURE_PING protocol
sudo -u jboss $EAP_HOME/wildfly/bin/jboss-cli.sh --echo-command \
'embed-host-controller --std-out=echo --domain-config=domain.xml --host-config=host-master.xml',\
':write-attribute(name=name,value=domain1)',\
'/profile=ha/subsystem=jgroups/stack=tcp:remove',\
'/profile=ha/subsystem=jgroups/stack=tcp:add()',\
'/profile=ha/subsystem=jgroups/stack=tcp/transport=TCP:add(socket-binding=jgroups-tcp,properties={ip_mcast=false})',\
"/profile=ha/subsystem=jgroups/stack=tcp/protocol=azure.AZURE_PING:add(properties={storage_account_name=\"${STORAGE_ACCOUNT_NAME}\", storage_access_key=\"${STORAGE_ACCESS_KEY}\", container=\"${CONTAINER_NAME}\"})",\
'/profile=ha/subsystem=jgroups/stack=tcp/protocol=MERGE3:add',\
'/profile=ha/subsystem=jgroups/stack=tcp/protocol=FD_SOCK:add(socket-binding=jgroups-tcp-fd)',\
'/profile=ha/subsystem=jgroups/stack=tcp/protocol=FD_ALL:add',\
'/profile=ha/subsystem=jgroups/stack=tcp/protocol=VERIFY_SUSPECT:add',\
'/profile=ha/subsystem=jgroups/stack=tcp/protocol=pbcast.NAKACK2:add(properties={use_mcast_xmit=false,use_mcast_xmit_req=false})',\
'/profile=ha/subsystem=jgroups/stack=tcp/protocol=UNICAST3:add',\
'/profile=ha/subsystem=jgroups/stack=tcp/protocol=pbcast.STABLE:add',\
'/profile=ha/subsystem=jgroups/stack=tcp/protocol=pbcast.GMS:add',\
'/profile=ha/subsystem=jgroups/stack=tcp/protocol=MFC:add',\
'/profile=ha/subsystem=jgroups/stack=tcp/protocol=FRAG3:add',\
'/profile=ha/subsystem=jgroups/channel=ee:write-attribute(name="stack", value="tcp")',\
'/server-group=main-server-group:write-attribute(name="profile", value="ha")',\
'/server-group=main-server-group:write-attribute(name="socket-binding-group", value="ha-sockets")',\
"/host=master/subsystem=elytron/http-authentication-factory=management-http-authentication:write-attribute(name=mechanism-configurations,value=[{mechanism-name=DIGEST,mechanism-realm-configurations=[{realm-name=ManagementRealm}]}])",\
"/host=master/interface=unsecure:add(inet-address=${HOST_VM_IP})",\
"/host=master/interface=management:write-attribute(name=inet-address, value=${HOST_VM_IP})",\
"/host=master/interface=public:add(inet-address=${HOST_VM_IP})"

# Save a copy of the domain.xml, later we need to share it with all host controllers
cp $EAP_HOME/wildfly/domain/configuration/domain.xml /tmp/domain.xml
```

The last stanza of output should look similar to the following. If it does not, troubleshoot and resolve the problem before continuing.

```bash
[domain@embedded /] /host=master/interface=public:add(inet-address=192.168.0.4 )
{
    "outcome" => "success",
    "result" => undefined,
    "server-groups" => undefined,
    "response-headers" => {"process-state" => "reload-required"}
}
02:05:55,019 INFO  [org.jboss.as] (MSC service thread 1-1) WFLYSRV0050: JBoss EAP 7.4.10.GA (WildFly Core 15.0.25.Final-redhat-00001) stopped in 28ms
```

Then, configure the JBoss server and setup the EAP service.

```bash
# Configure the JBoss server and setup EAP service
echo 'WILDFLY_HOST_CONFIG=host-master.xml' | sudo tee -a $EAP_RPM_CONF_DOMAIN

# Configure JBoss EAP management user
JBOSS_EAP_USER=jbossadmin
JBOSS_EAP_PASSWORD=Secret123456
sudo $EAP_HOME/wildfly/bin/add-user.sh  -u $JBOSS_EAP_USER -p $JBOSS_EAP_PASSWORD -g 'guest,mgmtgroup'
```

The output should look similar to the following.

```bash
Added user 'jbossadmin' to file '/etc/opt/rh/eap7/wildfly/standalone/mgmt-users.properties'
Added user 'jbossadmin' to file '/etc/opt/rh/eap7/wildfly/domain/mgmt-users.properties'
Added user 'jbossadmin' with groups guest,mgmtgroup to file '/etc/opt/rh/eap7/wildfly/standalone/mgmt-groups.properties'
Added user 'jbossadmin' with groups guest,mgmtgroup to file '/etc/opt/rh/eap7/wildfly/domain/mgmt-groups.properties'
```

Finally, start the EAP service.

```bash
# Start the JBoss server and setup EAP service
sudo systemctl enable eap7-domain.service

# Edit eap7-domain.services
sudo sed -i 's/After=syslog.target network.target/After=syslog.target network.target NetworkManager-wait-online.service/' /usr/lib/systemd/system/eap7-domain.service
sudo sed -i 's/Before=httpd.service/Wants=NetworkManager-wait-online.service \nBefore=httpd.service/' /usr/lib/systemd/system/eap7-domain.service

# Reload and restart EAP service
sudo systemctl daemon-reload
sudo systemctl restart eap7-domain.service

# Check the status of EAP service
systemctl status eap7-domain.service 
```

The output should look similar to the following.

```bash
● eap7-domain.service - JBoss EAP (domain mode)
   Loaded: loaded (/usr/lib/systemd/system/eap7-domain.service; enabled; vendor>
   Active: active (running) since Thu 2023-03-30 02:11:44 UTC; 5s ago
 Main PID: 3855 (scl)
    Tasks: 82 (limit: 20612)
   Memory: 232.4M
   CGroup: /system.slice/eap7-domain.service
           ├─3855 /usr/bin/scl enable eap7 -- /opt/rh/eap7/root/usr/share/wildf>
           ├─3856 /bin/bash /var/tmp/sclfYu7yW
           ├─3858 /bin/sh /opt/rh/eap7/root/usr/share/wildfly/bin/launch.sh /us>
           ├─3862 /bin/sh /opt/rh/eap7/root/usr/share/wildfly/bin/domain.sh --h>
           ├─3955 /usr/lib/jvm/jre/bin/java -D[Process Controller] -server -Xms>
           └─3967 /usr/lib/jvm/jre/bin/java -D[Host Controller] -Dorg.jboss.boo>

Mar 30 02:11:44 adminVM systemd[1]: Started JBoss EAP (domain mode).
```

Type **q** to exit the pager. Exit from the ssh connection by typing `exit`.

After starting the Red Hat JBoss EAP service, you will be able to access the management console via: `http://<adminVM_public_IP>:9990` in your web browser.  Sign in with the configured username: `jbossadmin` and password `Secret123456`.

:::image type="content" source="media/migrate-jboss-eap-to-vm-manually/adminconsole.png" alt-text="Screenshot of domain controller management console." lightbox="media/migrate-jboss-eap-to-vm-manually/adminconsole.png":::

Select **Runtime** tab. In the left navigation pane, select **Topology**. You should see that for now our cluster only contains one domain controller:

:::image type="content" source="media/migrate-jboss-eap-to-vm-manually/topology_only_with_admin.png" alt-text="Screenshot of cluster topology with domain controller only." lightbox="media/migrate-jboss-eap-to-vm-manually/topology_only_with_admin.png":::

#### Configure host controllers(worker nodes)

Use ssh to connect to the `mspVM1` as the `azureuser` user. Get the public IP address of the VM with this command.

```bash
az vm show --resource-group abc1110rg --name mspVM1 --show-details --query publicIps
```

These steps set up the host controller on `mspVM1`.

```bash
# environment variables
DOMAIN_CONTROLLER_PRIVATE_IP=<adminVM_private_IP>
HOST_VM_NAME=$(hostname)
HOST_VM_NAME_LOWERCASE=$(echo "${HOST_VM_NAME,,}")
HOST_VM_IP=$(hostname -I)

JBOSS_EAP_USER=jbossadmin
JBOSS_EAP_PASSWORD=Secret123456

# Save default domain configuration as backup
sudo -u jboss mv $EAP_HOME/wildfly/domain/configuration/domain.xml $EAP_HOME/wildfly/domain/configuration/domain.xml.backup

# Fetch domain.xml from domain controller
sudo -u jboss scp azureuser@${DOMAIN_CONTROLLER_PRIVATE_IP}:/tmp/domain.xml $EAP_HOME/wildfly/domain/configuration/domain.xml
```

You'll be asked for the password for the connection. For this example, the password is *Secret123456*.

Apply host controller changes to `mspVM1`.

```bash
# Setup host controller
sudo -u jboss $EAP_HOME/wildfly/bin/jboss-cli.sh --echo-command \
"embed-host-controller --std-out=echo --domain-config=domain.xml --host-config=host-slave.xml",\
"/host=${HOST_VM_NAME_LOWERCASE}/server-config=server-one:remove",\
"/host=${HOST_VM_NAME_LOWERCASE}/server-config=server-two:remove",\
"/host=${HOST_VM_NAME_LOWERCASE}/server-config=${HOST_VM_NAME_LOWERCASE}-server0:add(group=main-server-group)",\
"/host=${HOST_VM_NAME_LOWERCASE}/subsystem=elytron/authentication-configuration=slave:add(authentication-name=${JBOSS_EAP_USER}, credential-reference={clear-text=${JBOSS_EAP_PASSWORD}})",\
"/host=${HOST_VM_NAME_LOWERCASE}/subsystem=elytron/authentication-context=slave-context:add(match-rules=[{authentication-configuration=slave}])",\
"/host=${HOST_VM_NAME_LOWERCASE}:write-attribute(name=domain-controller.remote.username, value=${JBOSS_EAP_USER})",\
"/host=${HOST_VM_NAME_LOWERCASE}:write-attribute(name=domain-controller.remote, value={host=${DOMAIN_CONTROLLER_PRIVATE_IP}, port=9990, protocol=remote+http, authentication-context=slave-context})",\
"/host=${HOST_VM_NAME_LOWERCASE}/core-service=discovery-options/static-discovery=primary:write-attribute(name=host, value=${DOMAIN_CONTROLLER_PRIVATE_IP})",\
"/host=${HOST_VM_NAME_LOWERCASE}/interface=unsecured:add(inet-address=${HOST_VM_IP})",\
"/host=${HOST_VM_NAME_LOWERCASE}/interface=management:write-attribute(name=inet-address, value=${HOST_VM_IP})",\
"/host=${HOST_VM_NAME_LOWERCASE}/interface=public:write-attribute(name=inet-address, value=${HOST_VM_IP})"
```

The last stanza of output should look similar to the following. If it does not, troubleshoot and resolve the problem before continuing.

```bash
[domain@embedded /] /host=mspvm1/interface=public:write-attribute(name=inet-address, value=192.168.0.5 )
{
    "outcome" => "success",
    "result" => undefined,
    "server-groups" => undefined,
    "response-headers" => {"process-state" => "reload-required"}
}
02:58:59,388 INFO  [org.jboss.as] (MSC service thread 1-2) WFLYSRV0050: JBoss EAP 7.4.10.GA (WildFly Core 15.0.25.Final-redhat-00001) stopped in 58ms
```

Then, configure the JBoss server and setup EAP service.

```bash
echo 'WILDFLY_HOST_CONFIG=host-slave.xml' | sudo tee -a $EAP_RPM_CONF_DOMAIN

# Enable the JBoss server and setup EAP service
sudo systemctl enable eap7-domain.service

# Edit eap7-domain.services
sudo sed -i 's/After=syslog.target network.target/After=syslog.target network.target NetworkManager-wait-online.service/' /usr/lib/systemd/system/eap7-domain.service
sudo sed -i 's/Before=httpd.service/Wants=NetworkManager-wait-online.service \nBefore=httpd.service/' /usr/lib/systemd/system/eap7-domain.service

# Reload and restart EAP service
sudo systemctl daemon-reload
sudo systemctl restart eap7-domain.service

# Check the status of EAP service
systemctl status eap7-domain.service 
```

The output should look similar to the following.

```bash
● eap7-domain.service - JBoss EAP (domain mode)
   Loaded: loaded (/usr/lib/systemd/system/eap7-domain.service; enabled; vendor>
   Active: active (running) since Thu 2023-03-30 03:02:15 UTC; 7s ago
 Main PID: 9699 (scl)
    Tasks: 51 (limit: 20612)
   Memory: 267.6M
   CGroup: /system.slice/eap7-domain.service
           ├─9699 /usr/bin/scl enable eap7 -- /opt/rh/eap7/root/usr/share/wildf>
           ├─9700 /bin/bash /var/tmp/sclgJ1hRD
           ├─9702 /bin/sh /opt/rh/eap7/root/usr/share/wildfly/bin/launch.sh /us>
           ├─9706 /bin/sh /opt/rh/eap7/root/usr/share/wildfly/bin/domain.sh --h>
           ├─9799 /usr/lib/jvm/jre/bin/java -D[Process Controller] -server -Xms>
           └─9811 /usr/lib/jvm/jre/bin/java -D[Host Controller] -Dorg.jboss.boo>

Mar 30 03:02:15 mspVM1 systemd[1]: Started JBoss EAP (domain mode).
```

Type **q** to exit the pager. Exit from the ssh connection by typing `exit`.

Use ssh to connect to the `mspVM2` as the `azureuser` user. Get the public IP address of the VM with this command.

```bash
az vm show --resource-group abc1110rg --name mspVM2 --show-details --query publicIps
```

Repeat the above steps on `mspVM2`, and then exit the ssh connection by typing `exit`.

After two host controller are connected to `adminVM`, you should be able to see the cluster topology:

:::image type="content" source="media/migrate-jboss-eap-to-vm-manually/topology_with_cluster.png" alt-text="Screenshot of cluster topology with all hosts." lightbox="media/migrate-jboss-eap-to-vm-manually/topology_with_cluster.png":::

## Expose Red Hat JBoss EAP cluster with Azure Application Gateway

Now that you've created the Red Hat JBoss EAP cluster on Azure virtual machines, this section walks you through the process of exposing Red Hat JBoss EAP to the internet with Azure Application Gateway.

### Create the Azure Application Gateway

To expose Red Hat JBoss EAP to the internet, a public IP address is required. Create the public IP address and then associate an Azure Application gateway with it. Use [az network public-ip create](/cli/azure/network/public-ip#az-network-public-ip-create) to create it, as shown in the following example:

```azurecli
az network public-ip create \
    --resource-group abc1110rg \
    --name myAGPublicIPAddress \
    --allocation-method Static \
    --sku Standard
```

You'll add the backend servers to Application Gateway backend pool. Query backend IP addresses using the following commands. We will only have the host controllers(work nodes) configured as backend servers.

```azureclire
MSPVM1_NIC_ID=$(az vm show \
    --resource-group abc1110rg \
    --name mspVM1 \
    --query networkProfile.networkInterfaces'[0]'.id \
    --output tsv)
MSPVM1_IP=$(az network nic show \
    --ids ${MSPVM1_NIC_ID} \
    --query ipConfigurations'[0]'.privateIPAddress \
    --output tsv)
MSPVM2_NIC_ID=$(az vm show \
    --resource-group abc1110rg \
    --name mspVM2 \
    --query networkProfile.networkInterfaces'[0]'.id \
    --output tsv)
MSPVM2_IP=$(az network nic show \
    --ids ${MSPVM2_NIC_ID} \
    --query ipConfigurations'[0]'.privateIPAddress \
    --output tsv)
```

Next, create an Azure Application Gateway. The following example creates an application gateway with host controllers in the default backend pool.

```azurecli
az network application-gateway create \
    --resource-group abc1110rg \
    --name myAppGateway \
    --public-ip-address myAGPublicIPAddress \
    --location eastus \
    --capacity 2 \
    --http-settings-port 8080 \
    --http-settings-protocol Http \
    --frontend-port 80 \
    --sku Standard_V2 \
    --subnet jbossVMGatewaySubnet \
    --vnet-name myVNet \
    --priority 1001 \
    --servers ${MSPVM1_IP} ${MSPVM2_IP}
```

After the application gateway is created, you can see these new features:

- `appGatewayBackendPool` - A backend address pool includes the host controllers.
- `appGatewayBackendHttpSettings` - Specifies that port 80 and an HTTP protocol is used for communication.
- `rule1` - The default routing rule that's associated with *appGatewayHttpListener*.

> [!NOTE]
> This example sets up simple access to the Red Hat JBoss EAP servers with HTTP. If you want secure access, configure SSL/TLS termination by follow the instructions in [End to end TLS with Application Gateway](/azure/application-gateway/ssl-overview).

> This example exposes the host controllers at port 8080, we will deploy a sample application with database connection to the cluster in later steps.

## Connect Azure Database for PostgreSQL

This section shows you how to create a PostgreSQL instance on Azure and configure a connection to PostgreSQL on your Red Hat JBoss EAP cluster.

### Create an Azure Database for PostgreSQL instance

Use [az postgres server create](/cli/azure/postgres/server#az-postgres-server-create) to provision a PostgreSQL instance on Azure.

```azurecli
DATA_BASE_USER=jboss
DATA_BASE_PASSWORD=Secret123456

DB_SERVER_NAME="jbossdb$(date +%s)"
az postgres server create \
    --resource-group abc1110rg \
    --name ${DB_SERVER_NAME}  \
    --location eastus \
    --admin-user ${DATA_BASE_USER} \
    --ssl-enforcement Enabled \
    --admin-password ${DATA_BASE_PASSWORD} \
    --sku-name GP_Gen5_2
```

Allow access from Azure services:

```azurecli
# Write down the following names for later use
fullyQualifiedDomainName=$(az postgres server show \
    --resource-group abc1110rg --name ${DB_SERVER_NAME} \
    --query "fullyQualifiedDomainName" -o tsv)
name=$(az postgres server show \
    --resource-group abc1110rg --name ${DB_SERVER_NAME} \
    --query "name" -o tsv)

az postgres server firewall-rule create \
    --resource-group abc1110rg --server ${DB_SERVER_NAME} \
    --name "AllowAllAzureIps" --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
```

Create the database:
```azurecli
az postgres db create --resource-group abc1110rg --server ${DB_SERVER_NAME} --name testdb
```

### Install driver

We will be using JBoss management CLI to install the JDBC driver. For more information about JDBC drivers on Red Hat JBoss EAP, see [Installing a JDBC Driver as a JAR Deployment](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.4/html/configuration_guide/datasource_management#install_a_jdbc_driver_as_a_jar_deployment).

1. SSH to the `adminVM` by running the following command, you can skip this step if you have connection opened.

    ```bash
    ssh azureuser@$ADMIN_VM_PUBLIC_IP
    ```

2. Download JDBC driver, here we use `postgresql-42.5.2.jar`. For more informations about JDBC driver download locations, see [JDBC Driver Download Locations](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.4/html/configuration_guide/datasource_management#jdbc_driver_download_locations) provided by Red Hat.

    ```bash
    jdbcDriverName=postgresql-42.5.2.jar
    sudo curl --retry 5 -Lo /tmp/${jdbcDriverName} https://jdbc.postgresql.org/download/${jdbcDriverName}
    ```

3. Deploy JDBC driver use the following JBoss CLI command:

    ```bash
    sudo -u jboss $EAP_HOME/wildfly/bin/jboss-cli.sh --connect --controller=$(hostname -I) --echo-command \
    "deploy /tmp/${jdbcDriverName} --server-groups=main-server-group"
    ```
    
    The server log is located on `mspVM1` and `mspVM2` at `/var/opt/rh/eap7/lib/wildfly/domain/servers/mspvm1-server0/log/server.log`. If the deployment fails, examine this log file and resolve the problem before continuing.

### Configure the database connection for the Red Hat JBoss EAP cluster

You've started the database server, obtained the necessary resource ID and installed the JDBC driver, the steps in this section use the JBoss CLI to configure a datasource connection with the PostgreSQL instance created previously.

1. Open a terminal and SSH to the `adminVM` by running the following command:

    ```bash
    ssh azureuser@$ADMIN_VM_PUBLIC_IP
    ```

2. Create data source use the following JBoss CLI command:

    ```bash
    JDBC_DATA_SOURCE_NAME=dataSource-postgresql
    JDBC_JNDI_NAME=java:jboss/datasources/JavaEECafeDB
    DATA_SOURCE_CONNECTION_STRING=jdbc:postgresql://<database_full_qualified_domain_name>:5432/testdb
    DATA_BASE_USER=jboss@<database_server_name>
    DATA_BASE_PASSWORD=Secret123456
    JDBC_DRIVER_NAME=postgresql-42.5.2.jar
    
    sudo -u jboss $EAP_HOME/wildfly/bin/jboss-cli.sh --connect --controller=$(hostname -I) --echo-command \
    "data-source add --driver-name=${JDBC_DRIVER_NAME} --profile=ha --name=${JDBC_DATA_SOURCE_NAME} --jndi-name=${JDBC_JNDI_NAME} --connection-url=${DATA_SOURCE_CONNECTION_STRING} --user-name=${DATA_BASE_USER} --password=${DATA_BASE_PASSWORD}"
    ```

After the above steps, you've successfully configured a data source named `java:jboss/datasources/JavaEECafeDB`.

## Deploy Java EE Cafe sample application

This section shows how to deploy Java EE Cafe sample application to the Red Hat JBoss EAP cluster.

1. Following these instructions to build Java EE Cafe, assume you have a local environment with **Git** and **Maven** installed:

   Use the following command to clone the source code from GitHub:

   ```bash
   git clone https://github.com/Azure/rhel-jboss-templates.git
   ```

   Build the source code.

   ```bash
   mvn clean install --file eap-coffee-app/pom.xml
   ```

   This creates the file *eap-coffee-app/target/javaee-cafe.war*. You'll upload this file in the next step.

1. Open a web browser and go to the management console at `http://<adminVM_public_IP>:9990`, login with username `jbossadmin` and password `Secret123456`.

2. Upload the *javaee-cafe.war* to the **Content Repository**.

   1. From the **Deployments** tab of the Red Hat JBoss EAP management console, select **Content Repository** in the left navigation panel.
   1. Select the **Add** button and select **Upload Content**.
   :::image type="content" source="media/migrate-jboss-eap-to-vm-manually/upload_content.png" alt-text="Screenshot of Upload Content." lightbox="media/migrate-jboss-eap-to-vm-manually/upload_content.png":::
   1. Use the browser file chooser to select the *javaee-cafe.war* file.
   1. Select **Next**.
   1. Accept the defaults on the next screen and select **Finish**.
   1. Select **View content**.

4. Deploy an Application to the `main-server-group`.
    1. From **Content Repository**, select the `javaee-cafe.war`.
    1. Drop down the menue and and select the **Deploy** button.
    1. Select `main-server-group` as the server group for deploying `javaee-cafe.war`.
    1. Select **Deploy** to start the deployment. You should see a notice as shown here.
   :::image type="content" source="media/migrate-jboss-eap-to-vm-manually/successfully_deployed.png" alt-text="Screenshot of Upload Content." lightbox="media/migrate-jboss-eap-to-vm-manually/successfully_deployed.png":::

## Test the Red Hat JBoss EAP cluster configuration

You've now finished configuring the Red Hat JBoss EAP cluster and deploying the Java EE application to it. Use the following steps to access the application to validate all the settings.

1. Obtain the public IP address of the Azure Application Gateway
```bash
az network public-ip show --resource-group abc1110rg --name myAGPublicIPAddress --query '[ipAddress]' --output tsv
```
1. Open a web browser.
1. Navigate to the application with the URL `http://<gateway-public-ip-address>/javaee-cafe/`.  Don't forget the trailing slash.
1. Try to add/remove coffees.

## Clean up resources

To avoid Azure charges, you should clean up unnecessary resources. When you no longer need the Red Hat JBoss EAP cluster deployed on Azure VM, unregister the Red Hat JBoss EAP servers and remove Azure resources.

Run the following command to unregister the Red Hat JBoss EAP servers and VMs from Red Hat subscription management.

```azurecli
# Unregister domain controller
az vm run-command invoke    --resource-group ejb010329r     --name adminVM     --command-id RunShellScript     --scripts "sudo subscription-manager unregister"

# Unregister host controllers
az vm run-command invoke    --resource-group ejb010329r     --name mspVM1     --command-id RunShellScript     --scripts "sudo subscription-manager unregister"
az vm run-command invoke    --resource-group ejb010329r     --name mspVM2     --command-id RunShellScript     --scripts "sudo subscription-manager unregister"
```

Delete `abc1110rg` with the following command:

```azurecli
az group delete --name abc1110rg --yes --no-wait
```

## Next steps

Continue to explore options to run Red Hat JBoss EAP on Azure.

> [!div class="nextstepaction"]
> [Learn more about JBoss EAP on Azure](../ee/jboss-on-azure.md)
