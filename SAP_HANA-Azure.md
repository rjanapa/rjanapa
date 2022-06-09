<b>Run SAP HANA for Linux virtual machines in a scale-up architecture on Azure</b>

This reference architecture shows running SAP HANA in a highly available, scale-up environment that supports disaster recovery on Azure.

<img src="https://github.com/rjanapa/rjanapa/blob/main/SAP-HANA-Azure-Arch.png" width="500" length="500">

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
