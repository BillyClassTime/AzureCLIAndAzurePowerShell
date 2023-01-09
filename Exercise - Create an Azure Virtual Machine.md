# Exercise - Create an Azure Virtual Machine

## Task 1: Create a Linux virtual machine and install Nginx

Use the following Azure CLI commands to create a Linux VM and install Nginx. After your VM is created, you'll use the Custom Script Extension to install Nginx. The Custom Script Extension is an easy way to download and run scripts on your Azure VMs. It's just one of the many ways you can configure the system after your VM is up and running.

1. From Cloud Shell, run the followings commands to create a Linux VM: (You can choose Azure CLI or Azure PowerShell)

   Azure PowerShell:

   ```powershell
   Connect-AzAccount
   $rg = "RG-MV2023"
   New-AzResourceGroup -Name $rg -Location EastUs
   $ip = @{
       Name = 'myPublicIP'
       ResourceGroupName = $rg
       Location = 'eastus'
       Sku = 'Standard'
       AllocationMethod = 'Static'
       IpAddressVersion = 'IPv4'
       Zone = 1,2,3   
   }
   New-AzPublicIpAddress @ip
   $VMLocalAdminUser = "azureadmin";
   $VMLocalAdminSecurePassword = ConvertTo-SecureString "P455wrd.rd" -AsPlainText -Force;
   $Credential = New-Object System.Management.Automation.PSCredential ($VMLocalAdminUser, $VMLocalAdminSecurePassword);
   New-AzVM -ResourceGroupName $rg -Name newMV2023 -Credential $Credential -Image UbuntuLTS 
   $nic = Get-AzNetworkInterface -Name newMV2023 -ResourceGroupName $rg
   $pip = Get-AzPublicIpAddress -Name myPublicIP -ResourceGroupName $rg
   $vnet = Get-AzVirtualNetwork -Name newMV2023 -ResourceGroupName $rg
   $subnet = Get-AzVirtualNetworkSubnetConfig -Name newMV2023 -VirtualNetwork $vnet
   $nic | Set-AzNetworkInterfaceIpConfig -Name newMV2023 -PublicIPAddress $pip -Subnet $subnet
   $nic | Set-AzNetworkInterface
   Disconnect-AzAccount
   ```

   Azure CLI 

   ```azurecli
   az login 
   az group create --name RG-MV2023b --location EastUs
   ```
   Azure CLI con Bash:

   ```azurecli
   az vm create \
     --resource-group  RG-MV2023b\
     --name my-vm20 \
     --image UbuntuLTS \
     --admin-username azureadmin \
     --generate-ssh-keys
     --public-ip-sku Standard
   ```
   Azure CLI con PowerShell:  
   ```
   az vm create --resource-group  RG-MV2023b --name my-vm20 --image UbuntuLTS --admin-username azureadmin --generate-ssh-keys --public-ip-sku Standard
   ```

   Your VM will take a few moments to come up. You name the VM are **my-vm20** and **newMV2023** respectively. You use this name to refer to the VM in later steps.

2. Run the following commands to configure Nginx on your VM previously created:

   Azure Power Shell:

   ```powershell
   $Params = @{
       ResourceGroupName  = $rg
       VMName             = 'newMV2023'
       Name               = 'CustomScript'
       Publisher          = 'Microsoft.Azure.Extensions'
       ExtensionType      = 'CustomScript'
       TypeHandlerVersion = '2.1'
       Settings          = @{fileUris = @('https://raw.githubusercontent.com/MicrosoftDocs/mslearn-welcome-to-azure/master/configure-nginx.sh'); commandToExecute = './configure-nginx.sh'}
   }
   Set-AzVMExtension @Params
   ```

   Azure CLI con Bash:

   ```azurecli
   az vm extension set \
     --resource-group RG-MV2023b \
     --vm-name my-vm20 \
     --name customScript \
     --publisher Microsoft.Azure.Extensions \
     --version 2.1 \
     --settings '{"fileUris":["https://raw.githubusercontent.com/MicrosoftDocs/mslearn-welcome-to-azure/master/configure-nginx.sh"]}' \
     --protected-settings '{"commandToExecute": "./configure-nginx.sh"}'
   ```
   	Azure CLI con PowerShell:
   ```
   az vm extension set --resource-group RG-MV2023b --vm-name my-vm20 --name customScript --publisher Microsoft.Azure.Extensions --version 2.1 --settings "{'fileUris':['https://raw.githubusercontent.com/MicrosoftDocs/mslearn-welcome-to-azure/master/configure-nginx.sh']}" --protected-settings "{'commandToExecute': './configure-nginx.sh'}"
   ```

This command uses the Custom Script Extension to run a Bash script on your VM. The script is stored on GitHub. While the command runs, you can choose to [examine the Bash script](https://raw.githubusercontent.com/MicrosoftDocs/mslearn-welcome-to-azure/master/configure-nginx.sh) from a separate browser tab. To summarize, the script:

      1. Runs `apt-get update` to download the latest package information from the internet. This step helps ensure that the next command can locate the latest version of the Nginx package.
      2. Installs Nginx.
      3. Sets the home page, */var/www/html/index.html*, to print a welcome message that includes your VM's host name.

Las checks inside the virtual machines:

   1. Check the ```nginx``` server is ready using the following commands:

      ```
      sudo service nginx status
      sudo service nginx start
      sudo service nginx stop
      ```

   2. Checking IP address

      ```
      ip -h addr show
      ```

   3. You can not contact with 80 port because this is close on NSG

   4. You can check is 80 port is open using ```lsof``` command and netstat

      ```
      This lsof command is used to find the files and processes used by a user. The options used here are:
      
      -i: If no IP address is specified, this option selects the listing of all network files
      -P: inhibits the conversion of port numbers to port names for network files
      -n: inhibits the conversion of network numbers to host names for network files
      
      sudo lsof -i -P -n
      netstat -l
      ```

      

