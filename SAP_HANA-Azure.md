<b>Run SAP HANA for Linux virtual machines in a scale-up architecture on Azure</b>

This reference architecture shows running SAP HANA in a highly available, scale-up environment that supports disaster recovery on Azure.

<img src="https://github.com/rjanapa/rjanapa/blob/main/SAP-HANA-Azure-Arch.png" width="500" length="500">

<b>Azure Virtual Network</b><br>

The Azure Virtual Network service is used to define an isolated network in Azure. <br>

The virtual network can be used to host resources such as Azure virtual machines.<br>

The Azure virtual network gets assigned an address space which you specify when you create an Azure virtual network<br>

You can then add subnets to your Azure virtual network. This helps divide your network into more logical segments.

You could have one subnet named Subnet A in the virtual network to host your Web servers and another subnet to host the Database servers.

When you create a virtual machine in a virtual network, the virtual machine gets a Private IP address from the address space of the subnet is it launched in.

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

<b>Site-to-Site VPN Connection</b><br>

A Site-to-Site VPN connection is used to establish a secure connection between an on-premises network and an Azure network via the Internet.

On the on-premises side, you need to have a VPN device that can route traffic via the Internet onto the VPN gateway in Azure.

The VPN device can be a hardware device like a Cisco router or a software device (e.g., Windows Server 2016 running Routing and Remote services). 

The VPN device needs to have a publicly routable IP address.

The subnets in your on-premises network must not overlap with the subnets in your Azure virtual network.

The Site-to-Site VPN connection uses an IPSec tunnel to encrypt the traffic.

The VPN gateway resource you create in Azure is used to route encrypted traffic between your on-premises data center and your Azure virtual network.

<img src="https://github.com/rjanapa/rjanapa/blob/main/Site2SiteConnection.png" width="500" length="500">

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

<b>Azure Storage Accounts – Replication</b><br>

There are different replication techniques available to make your data highly available.

The different replication techniques available

1.	<b>Locally-redundant storage (LRS)</b> - Here data is replicated synchronously three times within a physical location in the primary region.

2.	<b>Zone-redundant storage (ZRS)</b> - Here data is replicated synchronously across three Azure availability zones in the primary region. This is good when you want to have data present even in the event of a data center failure.<br>

3.	<b>Geo-redundant storage (GRS)</b> - Here data is replicated synchronously three times in the primary region, then replicated asynchronously to the secondary region.<br>

4.	<b>Read access Geo-redundant storage (RA-GRS)</b> - Here data is replicated synchronously three times in the primary region, then replicated asynchronously to the secondary region. Here the data in the secondary region is also available for read-only purposes.<br>

<b>Azure SQL Database (Platform as a service)</b><br>

This is a service that allows you to create a managed Microsoft SQL Server database on the cloud. 

The advantages of using this service

1.	You don't have to manage the underlying infrastructure. This is managed by Azure.

2.	You have a variety of purchasing options

3.	You have automated backups. This reduces the burden of managing backups.

4.	It gives you a service level agreement of 99.99%

If you need to have more control over the database engine, then consider installing the SQL Server engine on an Azure virtual machine.

<b>High Availability</b><br>

This refers to technologies that can be used to minimize IT disruptions by ensuring applications and infrastructure is made fault tolerant.

Let's say that your application is hosted on a single virtual machine.

What happens if the virtual machine goes down for any reason, your application would not be available?

To make your application more redundant and more tolerant to failures, why not host your application on a collection of servers

Here even if one machine were to go down, you would still have the other one available. This makes your application more tolerant to infrastructure level failures.

<img src="https://github.com/rjanapa/rjanapa/blob/main/HighlyAvailableVM.png" width="500" length="500">

You can also increase the availability for your virtual machines by distributing them across Availability Zones or Availability Sets.

<b>Disaster Recovery</b><br>

This refers to the concept of minimizing IT disruptions by recovering them to another data center that could be located hundreds to miles away from the original data center hosting your application.

The following architecture diagram is an example of implementing disaster recovery.

Here your application is running on virtual machines in the West US region. Here the users are accessing your application.

At the same time, you might have the application hosted in another region (East US). The application might be in a shutdown state. This is only meant to be running if the primary region goes down for any reason.

Not let’s say there is a disaster in the West US region and all the data centers go down.

To minimize any disruption to your users, the requests to the application could now be redirected to the application in the East US region. So now you would start the application here and make sure all requests are routed to the secondary region.

<img src="https://github.com/rjanapa/rjanapa/blob/main/DR.png" width="500" length="500">

<img src="https://github.com/rjanapa/rjanapa/blob/main/DR_Regions.png" width="500" length="500">

<b>Azure Load Balancer</b><br>

The Azure Load balancer is used to distribute incoming network traffic to a backend group of servers.

This service helps increase the availability of your entire application architecture

Here the Load Balancer would take the incoming requests from the users and direct the requests to virtual machines running in an Azure virtual network.

If you have a web application running on the backend virtual machines, the requests would be distributed across the virtual machines by the Azure Load Balancer.

<img src="https://github.com/rjanapa/rjanapa/blob/main/AzureLoadBalancer.png" width="500" length="500">

<b>Azure Traffic Manager</b><br>

The Azure Traffic Manager service is a DNS-based traffic load balancer that distributes traffic across services that are distributed across different Azure regions.

The Traffic Manager service is used to direct client requests to the most appropriate service endpoint that is based on a traffic-routing method and the health of the endpoints.

Here we are assuming that a company has similar web applications, both are running using the Azure Web App service. One web application is running in the East US Region and the other is running in the West US Region.

1. Here we create a Traffic Manager profile and create two endpoints. Each endpoint points to each Azure Web app respectively. We assign a priority of 1 to the service endpoint attached to the Azure Web App running in the East US region and a priority of 2 to the other service endpoint.

2. Here users would make requests to the Traffic Manager service.

3. The requests could initially be directed to an Azure Web App located in the East US region, since there is a priority of 1 to the service endpoint attached to this endpoint.

4. Now let's say there is an issue with the web application running in the East US region, Azure Traffic Manager would then understand that there is an issue with the web application running in this region.

It would then start redirecting user requests to the second endpoint which has the Priority of 2.

Hence over here you are adding a higher availability to your architecture by ensuring that user requests are always adhered to by redirecting requests if the primary service fails for any reason.

<img src="https://github.com/rjanapa/rjanapa/blob/main/AzureTrafficManagerPriority1.png" width="500" length="500">

<img src="https://github.com/rjanapa/rjanapa/blob/main/AzureTrafficManagerPriority2.png" width="500" length="500">

And then we have the Geographic routing method wherein users would be directed to an endpoint based on their geographic location

<img src="https://github.com/rjanapa/rjanapa/blob/main/AzureTrafficManagerGeoRouting.png" width="500" length="500">

<b>Azure Content Delivery Network</b><br>

This is an ideal service to use for your web applications. If you need content to be distributed to users across the world for your web sites, then it’s ideal to use the Azure Content Delivery Network Service

Here the users are directed on various Edge servers by the Content Delivery Network service.

The Edge servers will get the content from your web site and cache frequently accessed content.

The Edge servers are located across the world, so it gives all users a seamless experience when it comes to accessing your web site.

<img src="https://github.com/rjanapa/rjanapa/blob/main/AzureCDN.png" width="500" length="500">

<b>Application Insights</b><br>

Points on this service

Application Performance Management service for web developers.

You can use this tool to monitor your applications.

It can help developers detect anomalies in the application.

It can help diagnose issues.

It can also help understand how users use your application.

It also helps you improve performance and usability of your application.

<b>How does it work</b><br>

You install a small instrumentation package within your application.

You can see the statistics of your application locally in Visual Studio as you run your application.

You can also use the Application Insights resource in Azure to monitor your application.

<b>What are the different aspects monitored by Application Insights?</b><br>

Request rates, the response times and failure rates – This is done at the page level.

Exception recorded by your application.

Page views and their load performance as reported from the user’s browser.

User and session counts.

Performance counters of the underlying Windows or Linux Machines.

Diagnostic trace logs from your application.

Any custom events or metrics that the developer writes themselves in the code.

<b>Azure Security Center</b><br>

This is an infrastructure security management system.

You can use this tool to improve the security of your Azure based resources and on-premises resources as well.

Azure Security Center has in-built support for services such as Azure virtual machines, Function Apps, Azure SQL Server databases.

You can also allow Azure Security Center to give recommendations on what to do for on-premises Windows and Linux servers.

On these servers, you need to ensure you install the Microsoft Monitoring agent.

This service also helps detect and prevent threats at an Infrastructure layer

<b>Azure Firewall</b><br>

This is a managed, cloud-based network security service that can be used to protect your network resources.

It has features such as Threat intelligence – This can filter incoming requests and alert or deny traffic from/to malicious IP addresses and domains.

The firewall itself has built-in high availability.

It can scale automatically based on network traffic flows.

Here you can ensure that all traffic from machines in an Azure virtual network flows via the Azure Firewall service.

<img src="https://github.com/rjanapa/rjanapa/blob/main/AzureFirewall.png" width="500" length="500">




