<b>System Design Generic Tips</b><br>

Step 1:<br> 
Functional Req<br>
Design Constraints<br>

Step 2: Define Microservices<br>

Step 3: Draw Logical Architecture<br>
Block Diagram for each Microservice<br>
Data/Logic flow between them<br>
Rules of Thumb:
1. If high volume of data needs to be pushed in near real time between two Microservices, use Pub-Sub. Pub-Sub is a Microservice of its own.<br>
2. If data needs to be pulled from Server to Client, use REST API.<br>
3. If data transfer is offline use Batch ETL (Extract Transform Load) Job.<br>

Step 4: Deep dive into each Microservice at a time<br>
Each Microservice consists of one or more tiers<br>
App Server Tier - Application Logic<br>
Cache Server Tier - For high througput data access and in-memory compute<br>
Storage Server Tier - Data Persistence<br>

<img src="https://github.com/rjanapa/rjanapa/blob/main/3-tier-arch-diagram.png" width="250"><br>

Step 4a<br>
For each microservice <br>
○ Identify Data Model (what data needs to be stored) to match the functional requirements<br>
○ Discuss how data will be stored in storage and cache tiers<br>
○ Propose APIs to match the functional requirements<br>
○ Propose workflow/algorithm for the the API in each tier<br>
○ Propose flow across tiers within the microservice<br>

Data Model<br>

API<br> 
Similar to CRUD operations<br>
create(v)<br>
read(k)<br>
update(k,v)<br>
delete(k)<br>

How data will be stored in storage and cache tiers e.g. HashMap<br>

Algorithm<br>

Step 4b<br>
For each microservice, Check whether each tier needs to scale<br>
● Need to scale for storage (storage and cache tiers)<br>
● Need to scale for throughput (CPU/IO)<br>
● Need to scale for API parallelization<br>
● Need to remove hotspots<br>
● Availability and Geo-distribution<br>
● Solve algebraically first and then put numbers<br>

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

Generic Tips<br>
Number of writes per second: when human generates a workload: 1000s to 10s of 1000s/sec<br>
Number of reads per second in a read heavy system: 100s of 1000s/sec<br>
Number of writes per second: when system is generating: 1 million per second<br>

Latency Numbers for Simple K-V workloads<br>
Application Server = 500 microseconds - 1 millisecond<br>
In-Memory Server = 1 - 3 millisecond<br>
Storage Server = 5 - 10 millisecond<br>

Step 4c<br>
○ Draw a generic distributed architecture per tier<br>
○ If app server tier and stateless, round robin requests <br>
○ If cache/storage tier, Partition the data into shards or buckets to suit requirements of scale<br>
○ Propose replication of shards<br>
○ Propose CP or AP (algorithms for CAP theorem)<br>

Two way mapping<br>
○ Data/Logic -> buckets/partitions/shards Done by the engineer<br>
○ Shards -> Server(s) Done by the cluster manager<br>
○ Helps in reducing metadata bloat<br>

