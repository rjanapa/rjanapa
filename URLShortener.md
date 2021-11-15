<b>URL Shortener</b><br>

<b>Step 1:</b><br> 
<b>Functional Requirements</b><br>
● Given a long URL generate a short URL<br>
● Given a short URL return long URL<br>
● Generate custom URL<br>
● TTL of Generated URL<br>
● Analytics<br>

<b>Design Constraints</b><br>
● Number of URL generated per second<br>
● Number of URL retrieved per second<br>
● Size of Short URL. Assume 7 character long string.<br>
● Characters in Short URL 0..9,a..z,A..Z<br>

<b>Step 2: Define Microservices</b><br>

<b>Step 3: Draw Logical Architecture</b><br>
Block Diagram for each Microservice<br>
Data/Logic flow between them<br>

<b>Step 4: Deep dive into each Microservice at a time</b><br>

<b>Step 4a</b>
For each microservice <br>
○ Data Model <br>
○ how data stored in storage and cache tiers<br>
○ API to match the functional requirements<br>
○ Workflow/algorithm for the the API in each tier<br>
○ Flow across tiers within the microservice<br>

<b>Data Model</b><br>
Short URL/Unique Id, Long URL, TTL, Creation Time<br>
k: Short URL/unique id<br>
v: Long URL, TTL, Creation Time<br>

<b>How data stored in storage and cache tiers:</b> HashMap<br>

<b>API</b><br> 
Similar to CRUD operations<br>
create(Long URL) -> create(v)<br>
read(Short URL) -> read(k)<br>
update(k,v)<br>
delete(k)<br>

create(Long URL)<br>
● Comes to App Tier<br>
● Send directly to Storage Tier<br>
● Generate unique id for long URL<br>
● Store unique id:long URL in Storage Tier<br>
● Store unique id:long URL in Cache Tier<br>
● Convert the unique id to a unique 7 character long string in the App Tier<br>

read(Short URL)<br>
● App Tier gets request<br>
● Converts Short URL back to unique id<br>
● Sent to Cache Tier<br>
● Cache Tier checks hashmap and returns if there is cache hit<br>
● Whenever there is a cache miss, hit the backend database. Whenever this happens, update the cache and pass the new entry to all the cache replicas. Each replica can update its cache by adding the new entry.

<b>Algorithm</b><br>
● Convert unique id to 7 character long string<br>
● 62 characters(0..9,a..z,A..Z), 7 positions = 62^7 = (2^6)^7 = 2^42 = (2^10)(2^10)(2^10)(2^10)(2^2) = 4 trillion<br>

<b>Step 4b</b>
For each microservice, Check whether each tier needs to scale<br>
■ Need to scale for storage (storage and cache tiers)<br>
■ Need to scale for throughput (CPU/IO)<br>
■ Need to scale for API parallelization<br>
■ Need to remove hotspots<br>
■ Availability and Geo-distribution<br>

<b>Storage Calculation</b><br>
Size of (k,v) pairs = A<br>
Number of lifetime (k,v) pairs = B<br>
Number of (k,v) pairs generated per sec = C<br>
Storage = (A * B)  or (C * Number of seconds in say 2 years * B) or (C * TTL (in seconds) * B)<br>
Cache = 20-30% of Storage<br>

<b>CPU Throughput</b><br>

<b>IO Throughput</b><br>

Availability 99.999%<br>

URL Shortner is human generated writes and read heavy system






