<b>Run SAP HANA for Linux virtual machines in a scale-up architecture on Azure</b>

This reference architecture shows running SAP HANA in a highly available, scale-up environment that supports disaster recovery on Azure.

<img src="https://github.com/rjanapa/rjanapa/blob/main/SAP-HANA-Azure-Arch.png" width="500" length="500">

<b>Azure Virtual Network</b><br>

The Azure Virtual Network service is used to define an isolated network in Azure. <br>

The virtual network can be used to host resources such as Azure virtual machines.<br>

The Azure virtual network gets assigned an address space which you specify when you create an Azure virtual network<br>

You can then add subnets to your Azure virtual network. This helps divide your network into more logical segments.

You could have one subnet named Subnet A in the virtual network to host your Web servers and another subnet to host the Database servers.

<b>When you create a virtual machine in a virtual network, the virtual machine gets a Private IP address from the address space of the subnet is it launched in.</b><br>

<b>Network Security Groups (NSG)</b><br>

These are used to filter network traffic to and from Azure resources in an Azure virtual network.

A network security group is attached to the network interface attached to the virtual machine.

A network security group consists of Inbound rules that are used to control the traffic inbound into a virtual machine

By default, all traffic into a virtual machine is DENIED.

You must explicitly add rules to allow traffic into a virtual machine

There are also outbound rules to control the traffic flowing out of the virtual machine. By default, all traffic outbound onto the Internet is allowed.

<b>Virtual Network Peering</b><br>

Virtual Network Peering is used to connect two Azure virtual networks together via the backbone network.

Azure supports connecting two virtual networks located in the same region or networks located across regions.

Once you enable virtual network peering between two virtual networks, the virtual machines can then communicate via their private IP addresses across the peering connection.

<b>Point-to-Site VPN Connection</b><br>

A Point-to-Site VPN connection is used to establish a secure connection between multiple client machines and an Azure virtual network via the Internet.

To implement a Point to Site VPN connection, you need to create a VPN Gateway in Azure.

<img src="https://github.com/rjanapa/rjanapa/blob/main/Point2SiteConnection.png" width="500" length="500">

<b>Availability Sets</b><br>

When you host your virtual machines in Azure, you sometimes need to cater to the following

1.	An unplanned event wherein the underlying infrastructure fails unexpectedly. The failures could be attributed to network failures, local disk failures or even rack failures.

2.	Planned maintenance events, wherein Microsoft needs to make planned updates to the underlying physical environment. In such cases, a reboot might be required on your virtual machine.

You can increase the availability of your application by making use of availability sets. 

<b>If you deploy two or more virtual machines in an Availability set, you will get a guarantee of virtual machine connectivity to at least one virtual machine 99.95% of the time.</b>

<b>Availability Zones</b><br>

1. This features help provides better availability for your application by protecting them from datacenter failures.

2. Each Availability zone is a unique physical location in an Azure region.

3. Each zone comprises of one or more data centers that has independent power, cooling, and networking

4. Hence the physical separation of the Availability Zones helps protect applications against data center failures

<b>Using Availability Zones, you can be guaranteed an availability of 99.99% for your virtual machines. You need to ensure that you have 2 or more virtual machines running across multiple availability zones</b>
