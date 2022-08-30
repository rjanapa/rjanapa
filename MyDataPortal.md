<b>My Data Portal</b><br>

The myData Portal (MDP) application is leveraged for analysis, data mining, ad-hoc 
reporting, research and used for implementing unstructured fast turn-around solutions 
to immediate business problems. mDP retains multiple key Digital Retail and Care Cross 
Channel support data in support of Wireline, Wireless, DTV, and OTT Call Center and 
Retail performance monitoring and analysis for use by Managers and Executives. 
MDP application is a solution dealing with data analysis and reporting key metrics to 
management enabling them to make important decisions for increasing agent productivity, 
enhancing customer service, reporting Customer Experience, Digital 1st, Clarify, myATT Zone, 
Snapshot, Call Driver, Outlier, Quicklinks, CDE, FRAO/MAO, IPBB and other related metrics.
Works with SSRS, Tableau, BusinessObjects, MicroStrategy, and PowerBI, domains are 
ITSERVICES, EMEA, AMERICAS, ATTAPA, and INTL.

<b>Risks/Blockers</b><br>

Ingest Tera data (2 TB of new data) from on premises into Cloud - priority 1 via 
ADF - Self Integration Runtime.

Throughput is poor at 115KB/sec. MSFT product group (PG) worked on this issue. 
It is when the table column count is very high. PG actively worked on the performance 
improvement but it may take some time to do tests and find solution.
had a call with Microsoft and ADF thru-put has reached 14 MBPS using 16 parallel threads
.32 parallel threads failed due to limitation in Teradata), 
required throughput is min 20 MBPS as per App Team requirements

Connectivity to SQL Server via ADF to be established - Required tables need to be created
first and then the Data export needs to happen. 

SQL Servers (IaaS) are exposed to Internet for connecting office 365 SharePoint : 
Connectivity needs established via Conexus Bastion Completed. Alignment to be done with         
MSFT Architect on alternative secure solution 
Testing between MDP SQL Server and Office 365 (bi-directional). 

55 strategic partner testing. 

Manually move data from Azure ADLS Gen2 to Snowflake through ADF - In Build Integration Runtime.

POC/PROD Voltage encryption programmatic (Java) encryption and decryption of data. 

Can disks such as I: and X: be added to Windows SQL Servers by latest 5.0 Stratum TF layers
 
ARB for S/W that need manual installation for MDP, VP approvals received and attached to RAID for ARB request      

<img src="https://github.com/rjanapa/rjanapa/blob/main/MDP-2022-08-30" width="500" length="500">
