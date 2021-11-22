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

<b>Step 2: Define Microservice</b><br>
CreatePasteMs <br>
ReadPasteMs <br>
DeletePasteMs <br>

<b>Step 3: Draw Logical Architecture: Block diagram for each Microservice, Data/Logic flow between them.</b>

<img src="https://github.com/rjanapa/rjanapa/blob/main/CreatePasteMicroservice.png" width="500" length="500">

<img src="https://github.com/rjanapa/rjanapa/blob/main/ReadPasteMs.png" width="500" length="500">

<b>Step 4: Deep dive into each Microservice</b> 

<b>Step 4a: For each Microservice – Data Model, How data is stored in Storage and Cache Tier, API, Workflow/Algorithm for API, Flow across Tiers</b>

<b>Step 4b: For each Microservice – check whether each tier needs to scale for storage, cache, throughput (CPU/IO), API parallelization, remove hotspots, Availability and Geo-Distribution</b>

<b>Step 4c: Draw a generic distributed architecture per tier</b>

<b>CreatePasteMs </b> <br>

<b>Handle a write request</b> <br>
Upon receiving a write-request, application server will generate a six-letter random string, which would serve as the key of the paste (if the user has not provided a custom key). The application server will then store the contents of the paste and the generated key in the database. After the successful insertion, the server can return the key to the user.

A standalone Key Generation Service (KGS) that generates random six letters strings beforehand and stores them in a database (let’s call it key-DB). Whenever we want to store a new paste, we will just take one of the already generated keys and use it. This approach will make things quite simple and fast since we will not be worrying about duplications or collisions. KGS will make sure all the keys inserted in key-DB are unique. KGS can use two tables to store keys, one for keys that are not used yet and one for all the used keys. As soon as KGS gives some keys to an application server, it can move these to the used keys table. KGS can always keep some keys in memory so that whenever a server needs them, it can quickly provide them. As soon as KGS loads some keys in memory, it can move them to the used keys table; this way we can make sure each server gets unique keys. If KGS dies before using all the keys loaded in memory, we will be wasting those keys. We can ignore these keys given that we have a huge number of them.

<b>Database Model</b>

<img src="https://github.com/rjanapa/rjanapa/blob/main/Paste.png" width="500" length="500">

<img src="https://github.com/rjanapa/rjanapa/blob/main/UserTable.png" width="500" length="500">

<b>API</b><br>
createPaste(api_dev_key, paste_data, custom_url=None user_name=None, paste_name=None, expire_date=None)

Parameters:
api_dev_key (string): The API developer key of a registered account. This will be used to throttle users based on their allocated quota.<br>
paste_data (string): Textual data of the paste.<br>
custom_url (string): Optional custom URL.<br>
user_name (string): Optional user name to be used to generate URL.<br>
paste_name (string): Optional name of the paste<br>
expire_date (string): Optional expiration date for the paste.<br>

Returns: (string)
A successful insertion returns the URL through which the paste can be accessed, otherwise, it will return an error code.

<img src="https://github.com/rjanapa/rjanapa/blob/main/CreatePasteMicroservice.png" width="500" length="500">

<b>ReadPasteMs </b> <br>

readPaste(api_dev_key, api_paste_key)

<b>Handle a paste read request</b> <br>
Upon receiving a read paste request, the application service layer contacts the datastore. The datastore searches for the key, and if it is found, it returns the paste’s contents. Otherwise, an error code is returned.

<img src="https://github.com/rjanapa/rjanapa/blob/main/ReadPasteMs.png" width="500" length="500">

<b>CleanupPasteMs </b> <br>

deletePaste(api_dev_key, api_paste_key)

A successful deletion returns ‘true’, otherwise returns ‘false’.


