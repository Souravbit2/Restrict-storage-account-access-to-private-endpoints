# Restrict-storage-account-access-to-private-endpoints
Create a private endpoint for your storage account within your Azure Virtual Network (VNet). This provides a private IP address for your storage account within your VNet.
Disable or restrict public network access to your storage account, so it's only accessible via the private endpoint.
Here are the detailed steps:
Prerequisites:
• An Azure Subscription.
• An Azure Storage Account (General-purpose v2 is recommended).
An Azure Virtual Network (VNet) with a subnet where you want to deploy the private endpoint.

**Step 1: Create a Virtual Network (VNet) and Subnet (if you don't have one)**
If you already have a VNet and subnet where you want to deploy the private endpoint, you can skip this step.
Go to the Azure portal.
Search for "Virtual networks" and select it.
Click "+ Create".
Fill in the Basics tab:
• Subscription: Select your Azure subscription.
• Resource Group: Create a new one or choose an existing one.
• Name: Give your VNet a meaningful name (e.g., WinVM-vnet).
• Region: Choose the region where your storage account and other resources will reside.
On the IP Addresses tab, configure your address space and add a subnet:
• Click "+ Add subnet".
• Subnet name: Give it a name (e.g., PrivateEndpointSubnet).
• Subnet address range: Define an appropriate CIDR range for your private endpoints (e.g., 10.0.0.0/24).
• Network security group (NSG): You can associate an NSG here, but for private endpoints, the NSG rules mainly apply to the source of the traffic, not the private endpoint itself. If you want to restrict traffic within your VNet to specific resources, you might apply NSG rules to the subnet hosting your clients, not the private endpoint subnet.
• Click "Add".
Proceed through Security, Tags (optional), and Review + create.
Click "Create".

<img width="1209" height="302" alt="image" src="https://github.com/user-attachments/assets/ee41a613-e2d6-4235-a5ba-f07f3fabe381" />

<img width="1282" height="558" alt="image" src="https://github.com/user-attachments/assets/ef87f664-7696-467b-9eeb-1dd0778b299b" />

**Step 2: Create a Private Endpoint for your Storage Account**
This step links your storage account to your VNet privately.
Go to your Azure Storage Account in the Azure portal.
In the left-hand menu, under "Security + networking", select "Networking".
Go to the "Private endpoint connections" tab.
Click "+ Private endpoint".
Basics Tab:
• Subscription: Auto-populated.
• Resource group: Auto-populated (should be the same as your storage account's RG, or choose another if you prefer to organize private endpoints separately).
• Name: Give your private endpoint a descriptive name (e.g., prvendp).
• Region: Auto-populated (should be the same as your storage account's region).
Resource Tab:
• Connection method: Leave as "Connect to an Azure resource in my directory".
• Subscription: Select your subscription.
• Resource type: Select "Microsoft.Storage/storageAccounts".
• Resource: Select your specific storage account from the dropdown.
• Target sub-resource: This is crucial. You need to create a separate private endpoint for each storage service you want to access (e.g., blob, file, queue, table, web). Select the service you need to restrict access to (e.g., blob). You'll repeat this step for other services if needed.
Virtual Network Tab:
• Virtual network: Select the VNet you created or chose in Step 1.
• Subnet: Select the subnet specifically designated for private endpoints (e.g., PrivateEndpointSubnet).
• Network policy for private endpoints: By default, this is disabled, which is generally what you want for private endpoint subnets. It means NSGs and route tables won't apply directly to the private endpoint's traffic flow within the subnet.
• Private IP configuration: Choose "Dynamically allocate IP address" (recommended) or "Statically allocate IP address" if you have a specific IP in mind.
DNS Tab:
• Integrate with private DNS zone: Select "Yes" (recommended). This will automatically create or link a private DNS zone (e.g., privatelink.blob.core.windows.net) and create an A record for your storage account within that zone. This allows clients in your VNet to resolve the storage account's public FQDN to its private IP address.
• If you choose "No" or use your own DNS servers, you'll need to manually configure DNS records to resolve the storage account's FQDN to the private endpoint's IP address.
Tags Tab: Add any desired tags (optional).
Review + create: Review your settings and click "Create".

<img width="1043" height="239" alt="image" src="https://github.com/user-attachments/assets/8f92c7d8-3f01-4ff5-9b30-9a282af3d4ef" />

<img width="1217" height="301" alt="image" src="https://github.com/user-attachments/assets/1353661f-022d-49cd-b9e3-1e3f1ef8160f" />

**Step 3: Restrict Public Network Access to the Storage Account**
This is the critical step to limit access only to private endpoints.
Go back to your Azure Storage Account in the Azure portal.
In the left-hand menu, under "Security + networking", select "Networking".
Go to the "Firewalls and virtual networks" tab.
Under "Public network access":
• To completely block all public access and allow only private endpoints: Select "Disabled".
• Important Note: If you select "Disabled", it means no traffic from the public internet, including your own public IP address or other Azure services that don't use private endpoints, will be able to reach your storage account. This is the most secure option if you want to enforce private endpoint-only access.
<img width="922" height="531" alt="image" src="https://github.com/user-attachments/assets/3cebecc4-fba1-4fc4-829e-13e63a3b4dd4" />


**Step 4: Verify Connectivity (from within your VNet)**
After setting up, it's essential to test that your clients can access the storage account via the private endpoint and that public access is blocked.
Deploy a Virtual Machine (VM) inside the same VNet and subnet where your client applications will reside (or in the PrivateEndpointSubnet if you are testing from there directly).
Access the VM (e.g., via Azure Bastion, RDP, or SSH).

<img width="1249" height="428" alt="image" src="https://github.com/user-attachments/assets/8155f600-0439-41c0-91a9-65bf3e41886d" />

Perform a DNS lookup on your storage account's FQDN prvendsg.blob.core.windows.net (e.g., yourstorageaccountname.blob.core.windows.net).
• You should see the private IP address of your private endpoint returned, not a public IP address.
• Example using nslookup:
Bash
nslookup yourstorageaccountname.blob.core.windows.net
Nslookup prvendsg.blob.core.windows.net (in my case)
The output should show a Non-authoritative answer with the Address being the private IP from your VNet.
Try to connect to the storage account from the VM using a tool like Azure Storage Explorer or an application. It should succeed.
Try to connect to the storage account from outside your VNet (e.g., from your local machine with no VPN/ExpressRoute connection to the VNet). It should fail, demonstrating that public access is blocked.

<img width="538" height="383" alt="image" src="https://github.com/user-attachments/assets/13957f7e-8f95-43a2-87d3-a75e60d92727" />









