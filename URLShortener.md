<b>URL Shortener</b><br>

<b>Step 1: Functional Requirements</b><br>
● Given a long URL generate a short URL<br>
● Given a short URL return long URL<br>
● Generate custom URL<br>
● TTL of Generated URL<br>

<b>Non-Functional Requirements:</b><br>
The system should be highly available. This is required because, if our service is down, all the URL redirections will start failing.<br>
URL redirection should happen in real-time with minimal latency.<br>
Shortened links should not be guessable (not predictable).<br>

<b>Extended Requirements:</b><br>
Analytics; e.g., how many times a redirection happened?<br>
Our service should also be accessible through REST APIs by other services.<br>

<b>Design Constraints</b><br>
● Number of URL generated per second<br>
● Number of URL retrieved per second<br>
● Size of Short URL. Assume 7 character long string.<br>
● Characters in Short URL 0..9,a..z,A..Z<br>

<b>Traffic Estimate</b><br>
Assuming, 500 million new URL shortenings per month<br>
read/write ratio: 100:1 <br>
100 * 500M => 50B redirections per month<br>
New URLs shortenings per second or Queries Per Second (QPS) for  the system = 500 million / (30 days * 24 hours * 3600 seconds) = 200 URL/s
Considering 100:1 read/write ratio, URLs redirections per second = 100 * 200 URLs/s = 20,000 URL/s <br>

<b>Storage Estimate</b><br>
Assume store every URL shortening request for 5 years. <br>
For 500M new URLs every month, the total number of objects = 500 million x 5 x 12 = 30000 million = 30B<br>
Assume each store object = 500 bytes<br>
30B X 500 bytes = 15000B bytes = 15,000 x 1000,000,000 = 15 TB<br>
1000 bytes = 1 KB<br>
1000 KB = 1 MB<br>
1000 MB = 1 GB<br>
1000 GB = 1 TB<br>

<b>What kind of database?</b><br> 
Since storing billions of rows, and no need to use relationships between objects => NoSQL store like DynamoDB, Cassandra or Riak . <br>
A NoSQL DB is easier to scale.<br>

<b>Bandwidth Estimate</b><br>
For write request 200 URL/second = 200 x 500 bytes = 100,000 bytes / second = 100 KB/second
For read request with 100:1 read:write ratio = 100 KB/second x 100 = 10,000 KB/second = 10 MB/second

<b>Memory Estimate</b><br>
Follow 80:20 rule, 20% of URL generate 80% of the traffic<br>
For Read Requests per second = 20,000 URL/second<br>
For Read Requests per day = 20,000 URL/second X 24 hours x 60 minutes x 60 seconds = 20,000 x 86,400 = 20,000 x 100,000 = 2000,000,000 = 2B requests/day<br>
Cache 20% of read requests per day = 2B x 500 bytes x 20% = 1000B bytes x 20% = 1000,000,000,000 bytes x 20% =  1 TB x 20% = 1000 GB x 0.2 = 200 GB<br>

<b>Step 2: Define Microservices</b><br>
CreateLongURLMicroservice<br>
ReadShortURLMicroservice<br>
UpdateURLMicroservice<br>
DeleteURLMicroservice<br>

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

<img src="https://github.com/rjanapa/rjanapa/blob/main/URLTable.png" width="250"> <img src="https://github.com/rjanapa/rjanapa/blob/main/UserTable.png" width="250"><br>

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
● 62 characters(0..9,a..z,A..Z), 7 positions = 62^7 ~ 64^7 = (2^6)^7 = 2^42 = (2^10)(2^10)(2^10)(2^10)(2^2) = 4 trillion<br>

compute a unique hash (e.g., MD5 or SHA256, etc.) of the given URL. <br>
encode the hash for display. <br>
The encoding could be base36 ([a-z ,0-9]) or <br>
base62 ([A-Z, a-z, 0-9]) and <br>
add ‘+’ and ‘/’ to use Base64 encoding.<br>

There are issues with the encoding:<br>
● If multiple users enter the same URL, they can get the same shortened URL.<br>
● If parts of the URL are URL-encoded e.g., http://github.com/mypage.php?id=systemdesign, and http://github.com/mypage.php%3Fid%3Dsystemdesign are identical except for the URL encoding.<br><br>
Workaround for the issues: Append an increasing sequence number to each input URL to make it unique and then generate its hash. There is no need to store this sequence number in the databases, though. Possible problems with this approach ia an ever-increasing sequence number possibly resulting in overflow. Appending an increasing sequence number also impact the performance of the service. <br>

Another solution could be to append the user id (which should be unique) to the input URL. However, if the user has not signed in, we would have to ask the user to choose a uniqueness key. Even after this, if we have a conflict, we have to keep generating a key until we get a unique one.<br>

<img src="https://github.com/rjanapa/rjanapa/blob/main/URLShorteningRequestFlow.png" width="250"> <img src="https://github.com/rjanapa/rjanapa/blob/main/UserTable.png" width="250"><br>

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

<b>Availability 99.999%</b><br>

URL Shortner is human generated writes and read heavy system<br>

<b>Step 4c:</b> Draw a generic distributed architecture per tier<br>

<b>Sharding (for both cache and storage) </b><br>
○ Horizontal sharding - partitioning by key<br>
○ Can use either a hash function or a range function<br>
○ [0 - 512 million] -> Shard 0 -> Server A, C, E (replication factor of 3)<br>
○ [512 million - 1 billion] -> Shard 1 -> Servers B, D, A<br>
○ 8000 shards<br>







