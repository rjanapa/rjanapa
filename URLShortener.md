<b>URL Shortener</b><br>

<b>Step 1:</b><br> 
Functional Req<br>
● Given a long URL generate a short URL<br>
● Given a short URL return long URL<br>
● Generate custom URL<br>
● TTL of Generated URL<br>
● Analytics<br>

Design Constraints<br>
● Number of URL generated per second<br>
● Number of URL retrieved per second<br>
● Size of Short URL. Assume 7 to start with.<br>
● Characters in Short URL 0..9,a..z,A..Z<br>

<b>Step 2: Define Microservices</b><br>

<b>Step 3: Draw Logical Architecture</b><br>
Block Diagram for each Microservice<br>
Data/Logic flow between them<br>

<b>Step 4: Deep dive into each Microservice at a time</b><br>

<b>Step 4a</b><br>
For each microservice <br>
○ Identify Data Model <br>
○ Discuss how data will be stored in storage and cache tiers<br>
○ Propose APIs to match the functional requirements<br>
○ Propose workflow/algorithm for the the API in each tier<br>
○ Propose flow across tiers within the microservice<br>

Data Model<br>
Short URL/Unique Id, Long URL, TTL, Creation Time<br>
k: Short URL/unique id<br>
v: Long URL, TTL, Creation Time<br>

How data will be stored in storage and cache tiers: HashMap<br>

API<br> 
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
● Converts Short URL back to unique id
● Sent to Cache Tier<br>
● Cache Tier checks hashmap and returns if there is cache hit<br>
● Whenever there is a cache miss, hit the backend database. Whenever this happens, update the cache and pass the new entry to all the cache replicas. Each replica can update its cache by adding the new entry.

Algorithm<br>
● Convert unique id to 7 character long string<br>
● 62 characters(0..9,a..z,A..Z), 7 positions = 62^7 = (2^6)^7 = 2^42 = (2^10)(2^10)(2^10)(2^10)(2^2) = 4 trillion<br>

<b>Step 4b</b><br>
For each microservice, Check whether each tier needs to scale<br>

Storage Calculation<br>
Size of (k,v) pairs = A<br>
Number of lifetime (k,v) pairs = B<br>
Number of (k,v) pairs generated per sec = C<br>
Storage = (A * B)  or (C * Number of seconds in say 2 years * B) or (C * TTL (in seconds) * B)<br>
Cache = 20-30% of Storage<br>

CPU Throughput<br>
Number of API calls that App Tier need to handle = Y<br>
Latency of API call = X millisecond = X/1000 seconds  =>> X . . X . . X . . X .. X . . X<br>
For a single thread nX = 1000 milliseconds<br>
For a single thread n = 1000/X operations per second<br>
Number of API handled by a single thread = 1000/X operations/second<br>
Number of concurrent threads in a commodity server typically 100 ~ 200 = Z<br>
Number of API handled by Z threads = (1000 * Z)/X operations/second<br>
A Server operating at 30-40% capacity<br>
Number of API handled by Z threads in one Server = (300 * Z)/X operations/second<br>

IO Throughput<br>
A single server can provide IO throughput (Medium spinning disk) = 100 - 200 MB/second = Z<br>
A single server can provide IO throughput (Medium Flash SSD) = 1 - 2 GB/second = Z<br>
Total I/O required by your system = Y MB/second<br>
Total # of servers = Y/Z<br>

Availability 99.999%<br>

Latency Numbers for Simple K-V workloads<br>
Application Server = 500 microseconds - 1 millisecond<br>
In-Memory Server = 1 - 3 millisecond<br>
Storage Server = 5 - 10 millisecond<br>

URL Shortner is human generated writes and read heavy system






