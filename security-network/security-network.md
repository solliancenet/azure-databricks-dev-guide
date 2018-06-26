# Azure Databricks Security and Networking

start here

## Perimeter Security

From a high level, the Azure Databricks perimeter security...

### Deployed in its own VNET

Now let's take a deeper look under the covers. When you create an Azure Databricks service, you may decide to use the default VNET or...

### NSG Rules

About those NSG Rules...

## VNET Pairing

Look at the info below...  https://docs.azuredatabricks.net/administration-guide/cloud-configurations/azure/vnet-peering.html

### Custom DNS Resolvers

Oh that DNS...

### Private DNS Configs

Blah...

## On-premise connectivity

Blah...

### Express Route

See info below -   
https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/hybrid-networking/expressroute

### Other Information

Q:Can I use my own keys for local encryption?  
A:In the current release, using your own keys from Azure Key Vault is not supported.

Q:Can I use Azure virtual networks with Databricks?
A:A new virtual network is created as part of Databricks provisioning. In this release, you cannot use your own Azure virtual network.

Q:How do I access Azure Data Lake Store from a notebook?
A: In Azure Active Directory (Azure AD), provision a service principal, and record its key.
Assign the necessary permissions to the service principal in Data Lake Store.
To access a file in Data Lake Store, use the service principal credentials in Notebook.

## More Information
 - Connect [on-premises network to Azure using Express Route](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/hybrid-networking/expressroute)
 - Connect via [a Site-to-Site connect](https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal)
 - List of Azure Databricks [regions](https://docs.azuredatabricks.net/administration-guide/cloud-configurations/regions.html)
 - For More: [Common Questions](https://docs.microsoft.com/en-us/azure/azure-databricks/frequently-asked-questions-databricks)
 - How to [connect using the Azure cloud shell](https://docs.microsoft.com/en-us/azure/azure-databricks/databricks-cli-from-azure-cloud-shell)
 - How to [connect to data sources](https://docs.microsoft.com/en-us/azure/azure-databricks/databricks-connect-to-data-sources)
 - How to [connect to Excel, R or Python](https://docs.microsoft.com/en-us/azure/azure-databricks/connect-databricks-excel-python-r)


 ---- integrate here

 Virtual Network Peering
Virtual network (VNet) peering allows the virtual network in which your Azure Databricks resource is running to peer with another Azure virtual network. Traffic between virtual machines in the peered virtual networks is routed through the Microsoft backbone infrastructure, much like traffic is routed between virtual machines in the same virtual network, through private IP addresses only. For an overview of Azure VNet peering, see Microsoft Azure Virtual network peering.

This topic shows you how to peer an Azure Databricks VNet with an Azure VNet and provides references to information about how to connect an on-premise VNet to an Azure VNet.

For information about how to manage Azure VNet peering, see Create, change, or delete a virtual network peering. However, do not follow the steps in that topic to create a VNet peering between an Azure Databricks VNet and an Azure VNet; follow the instructions below.

In this topic:

Peer a Databricks virtual network to a remote virtual network
Connect an on-premises virtual network to an Azure virtual network
Peer a Databricks virtual network to a remote virtual network
Peering a Databricks virtual network to an Azure VNet involves two steps:

Step 1: Add remote virtual network peering to Databricks virtual network
Step 2: Add Databricks virtual network peer to remote virtual network
Step 1: Add remote virtual network peering to Databricks virtual network
In the Azure Portal, click an Azure Databricks Service resource.

In the Settings section of the sidebar, click the Virtual Network Peering tab.

Click the + Add Peering button.

../../../_images/azure-virtual-network-peering1.png
In the Name field, enter the name for the peering virtual network.

Depending on the information you have about the remote virtual network, do one of the following:

You know the resource ID of the remote virtual network:

Select the I know my Resource ID checkbox.

In the Resource ID text box, paste in the remote virtual network resource ID.

../../../_images/azure-add-peering1a.png
You know the name of the remote virtual network:

In the Subscription drop-down, select a subscription.

In the Virtual network drop-down, select the remote virtual network.

../../../_images/azure-add-peering1b.png
In the Configuration section, specify the configuration of the peering. See Create a peering for information about the configuration fields.

In the Databricks Virtual Network Resource Id section, copy the resource ID.

Click Add. The virtual network peering is deployed.

Step 2: Add Databricks virtual network peer to remote virtual network
In the Azure Portal sidebar, click Virtual networks.

Search for the virtual network resource to peer with the Databricks virtual network and click the resource name.

In the Settings section of the sidebar, click the Peerings tab.

Click the + Add button.

../../../_images/azure-virtual-network-peering2.png
In the Name field, enter a name for the peering virtual network.

For the peering details, Virtual network deployment model must be Resource manager.

Select the I know my resource ID checkbox.

In the Resource ID text box, paste in the Databricks virtual network resource ID copied in Step 1.

../../../_images/azure-add-peering2.png
In the Configuration section, specify the configuration of the peering. See Create a peering for information about the configuration fields.

Click OK. The virtual network peering is deployed.

Connect an on-premises virtual network to an Azure virtual network
To connect an on-premises network to an Azure VNet, follow the steps in Connect an on-premises network to Azure using ExpressRoute.

To create a site-to-site VPN gateway connection from your on-premises network to an Azure VNet, follow the steps in Create a Site-to-Site connection in the Azure portal.

 
