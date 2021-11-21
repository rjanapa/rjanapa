Pastebin enable users to store plain text over the web and generates unique URL to access the uploaded content over the internet.

<b>Step 1: Functional Requirements, Non-Functional Requirements, Extended Requirements, Design Considerations</b>

<b>Functional Requirements</b>
Users should be able to upload or paste text and get a unique URL to access it
Data and links expire after a certain timeperiod and users should be able to specify expiration time
Users should optionally be able to pick a custom alias for their paste

<b>Non-Functional Requirements</b>
The system should be highly reliable. The uploaded content should not be lost.
The system should be highly availalable. 
The users should be able to access the content in real-time with minimum latency
paste links should not be predictable

<b>Extended Requirements</b>
Analytics - how many times a paste is accessed
The URL should be accessible via REST API

<b>Design Considerations</b>
Limit on the amount of text user can paste at a time. Limit users not to have Pastes bigger than 10MB.

<b>Capacity estimations and Constraints</b>
The service will be read heavy. There will be more reads compared to writes.
Assume 5:1 Read:Write

<b>Traffic Estimates</b>  <br>
Suppose 5 million reads/day  <br>
New pastes per second = 1 million / 24 x 60 x 60 = 12 paste / second  <br>
Paste reads per second = 60 / second

<b>Storage Estimates</b>  <br>
Suppose each paste 10 KB
Storage for 1 million paste day = 10KB x 1000,000 = 10,000 bytes x 1000,000 = 10 GB / day <br>
To store this data for 10 years = 10 GB x 365 x 10 = 36500 GB =  36.5 TB  <br>
Assume a 70% capacity model (don’t use more than 70% of our total storage capacity at any point)  <br>
=> storage needs = 36,5 TB X 100/70 = 52 TB
 
1 million pastes per day
Number of pastes in 10 years = 1 million x 365 x 10 = 3650 x 1000,000 = 3.65 billion pastes   <br>
Generate and store keys to uniquely identify these pastes.   <br>
If one use base64 encoding ([A-Z, a-z, 0-9, ., -]), one would need six letters strings => 64^6 ~= 68.7 billion unique strings  <br>
If it takes one byte to store one character, total size required to store 3.6B keys = 3.6B * 6 => 22 GB  

<b>Bandwidth estimates</b>  <br>
For write requests, 12 new pastes per second, => 12 x 10 KB = 120KB of ingress per second.  <br>
For read request, 60 requests per second. => total data egress (sent to users) = 60 * 10KB => 600 KB / second => 0.6 MB / second

<b>Memory estimates: </b>  <br>
Cache some of the hot pastes that are frequently accessed.   <br>
Following the 80-20 rule, meaning 20% of hot pastes generate 80% of traffic, we would like to cache these 20% pastes.  <br>
Since we have 5M read requests per day, to cache 20% of these requests = 0.2 * 5M * 10KB ~= 10 GB

<b>Step 2: Define Microservice</b>
AddPasteMicroservice
GetPasteMicroservice
DeletePasteMicroservice

<b>Step 3: Draw Logical Architecture: Block diagram for each Microservice, Data/Logic flow between them.</b>

<b>Step 4: Deep dive into each Microservice</b>

<b>Step 4a: For each Microservice – Data Model, How data is stored in Storage and Cache Tier, API, Workflow/Algorithm for API, Flow across Tiers</b>

<b>Step 4b: For each Microservice – check whether each tier needs to scale for storage, cache, throughput (CPU/IO), API parallelization, remove hotspots, Availability and Geo-Distribution</b>

<b>Step 4c: Draw a generic distributed architecture per tier</b>

