<b>Instagram</b><br>

Instagram is a social networking service that enables its users to upload and share their photos and videos with other users. <br>
Instagram users can choose to share information either publicly or privately. <br>
Anything shared publicly can be seen by any other user, whereas privately shared content can only be accessed by the specified set of people. <br>
Instagram also enables its users to share through many other social networking platforms such as Facebook <br>
A user can follow other users. <br>
The ‘News Feed’ for each user will consist of top photos of all the people the user follows<br>

<b>Step 1: Functional Requirements, Non-Functional Requirements, Extended Requirements, Design Constraints</b><br>

<b>Functional Requirements</b><br>

Users should be able to upload/download/view photos.<br>
Users can perform searches based on photo/video titles.<br>
Users can follow other users.<br>
The system should generate and display a user’s News Feed consisting of top photos from all the people the user follows.<br>

<b>Non-functional Requirements</b><br>

Our service needs to be highly available.<br>
The acceptable latency of the system is 200ms for News Feed generation.<br>
The system should be highly reliable; any uploaded photo or video should never be lost.<br>

<b>Design Considerations</b><br>

The system would be read-heavy, so system should retrieve photos quickly.<br>
Low latency is expected while viewing photos.<br>
Users can upload as many photos as they like; therefore, efficient management of storage is a crucial factor.<br>
Data should be 100% reliable. If a user uploads a photo, the system should guarantee that it will never be lost.<br>

<b>Step 2: Define Microservice</b><br>

Upload Image Microservice

View Image Microservice -> Download Image Microservice

Search Image Microservice -> Download Image Microservice

<b>Step 3: Draw Logical Architecture: Block diagram for each Microservice, Data/Logic flow between them.</b><br>

<b>High Level System Design Diagram</b><br>

<img src="https://github.com/rjanapa/rjanapa/blob/main/Instagram-high-level-diagram.png" width="500" length="500">

<b>Component Design</b><br>

Photo uploads can be slow as they have to go to the disk, whereas reads will be faster, especially if they are being served from cache.

Uploading users can consume all the available connections, as uploading is a slow process. This means that ‘reads’ cannot be served if the system gets busy with all the ‘write’ requests. Web servers have a connection limit before designing our system. Assume that a web server have a maximum of 500 connections at any time, then it can’t have more than 500 concurrent uploads or reads. To handle this bottleneck, split reads and writes into separate services and have dedicated servers for reads and writes to ensure uploads don’t hog the system.

Separating photos’ read and write requests allow to scale and optimize each of these operations independently.

<b>Reliability and Redundancy</b><br>
Losing files is not an option for our service. Therefore, store multiple copies of each file so that if one storage server dies, one can retrieve the photo from the other copy present on a different storage server.<br>

This same principle also applies to other components of the system. If one wants to have high availability of the system, one need to have multiple replicas of services running in the system so that even if a few services die down, the system remains available and running. Redundancy removes the single point of failure in the system.<br>

If only one instance of a service is required to run at any point, one can run a redundant secondary copy of the service that is not serving any traffic, but it can take control after the failover when the primary has a problem.

Creating redundancy in a system can remove single points of failure and provide a backup or spare functionality if needed in a crisis. For example, if there are two instances of the same service running in production and one fails or degrades, the system can failover to the healthy copy. Failover can happen automatically or require manual intervention.<br>

<b>Step 4: Deep dive into each Microservice</b><br>

Step 4a: For each Microservice – Data Model, How data is stored in Storage and Cache Tier, API, Workflow/Algorithm for API, Flow across Tiers

Step 4b: For each Microservice – check whether each tier needs to scale for storage, cache, throughput (CPU/IO), API parallelization, remove hotspots, Availability and Geo-Distribution

Step 4c: Draw a generic distributed architecture per tier

<b>Upload Image Microservice</b><br>

<b>Data Model, How data is stored in Storage and Cache Tier</b><br>
Store data about users, their uploaded photos, and the people they follow. <br>
The Photo table store all data related to a photo. <br>
Index on (PhotoID, CreationDate) since need to fetch recent photos first.<br>

<img src="https://github.com/rjanapa/rjanapa/blob/main/Instagram-DB-Tables.png" width="500" length="500">

A straightforward approach for storing the above schema would be to use an RDBMS like MySQL since we require joins. But RDBMS come with their challenges, especially when one needs to scale them. <br>

Store photos in a distributed file storage like HDFS or S3.<br>

Store the above schema in a distributed key-value store to enjoy the benefits offered by NoSQL. All the metadata related to photos go to a table where the ‘key’ is‘PhotoID’ and the ‘value’ is an object containing (PhotoLocation, UserLocation, CreationTimestamp, etc).

If one go with a NoSQL database, need an additional table  ‘UserPhoto’ to store the relationships between users and photos to know who owns which photo.  <br>

Also need a table ‘UserFollow’ to store the list of people a user follows. <br>

For both the tables, Use a wide-column datastore like Cassandra. For the ‘UserPhoto’ table, the ‘key’ would be ‘UserID’, and the ‘value’ would be the list of ‘PhotoIDs’ the user owns, stored in different columns. Have a similar scheme for the ‘UserFollow’ table.

<b>Data Size Estimation</b><br>

Photo: Each row in Photo’s table = 284 bytes: <br>

PhotoID (4 bytes) + UserID (4 bytes) + PhotoPath (256 bytes) + PhotoLatitude (4 bytes) + PhotoLongitude(4 bytes) + UserLatitude (4 bytes) + UserLongitude (4 bytes) + CreationDate (4 bytes) = 284 bytes<br>

If 2M new photos get uploaded every day, we will need 0.5GB of storage for one day:<br>

2M * 284 bytes ~= 2,000,000 x 284 bytes = 500 MB = 0.5GB per day<br>

For 10 years we will need 1.88TB of storage i.e 0.5 GB X 400 (365) x 10  = 2000 GB = 2 TB<br>

User: Assume each “int” and “dateTime” is four bytes, each row in the User’s table = 68 bytes <br>

UserID (4 bytes) + Name (20 bytes) + Email (32 bytes) + DateOfBirth (4 bytes) + CreationDate (4 bytes) + LastLogin (4 bytes) = 68 bytes<br>

Suppose 500 million users, one need 50GB of total storage.<br>

500 million * 68 ~= 500 x 1,000,000 x 100 = 50,000,000,000 = 50GB<br>

UserFollow: Each row in the UserFollow table consists of 8 bytes. <br>

Suppose 500 million users and on average each user follows 500 users. One would need 2.5TB of storage for the UserFollow table:<br>

500 million users * 500 followers * 8 bytes = 2500,000,000,000 = 2500GB = 2.5TB<br>

Total space required for all tables for 10 years = 5TB:<br>

50GB + 2TB + 2.5TB = 5TB<br>

<b>Data Sharding</b><br>

<b>a. Partitioning based on UserID </b><br>
Assume shard based on the ‘UserID’ so that keep all photos of a user on the same shard. If one DB shard is 1TB, one will need 5 shards to store 5TB of data. Assume, for better performance and scalability, keep 10 shards.

So find the shard number by UserID % 10 and then store the data there. To uniquely identify any photo in the system, append the shard number to each PhotoID.

How can one generate PhotoIDs? Each DB shard can have its own auto-increment sequence for PhotoIDs, and since one will append ShardID with each PhotoID, it will make it unique throughout the system.

What are the different issues with this approach?

Several people follow hot users e.g. celebrities, and a lot of other people see any photo they upload.
Some users will have a lot of photos compared to others, thus making a non-uniform distribution of storage.
What if one cannot store all pictures of a user on one shard? If one distribute photos of a user onto multiple shards, will it cause higher latencies?
Storing all photos of a user on one shard can cause issues like unavailability of all of the user’s data if that shard is down or higher latency if it is serving high load etc.

<b>b. Partitioning based on PhotoID </b><br>
If one can generate unique PhotoIDs first and then find a shard number through “PhotoID % 10”, the above problems will be solved and there is no need to append ShardID with PhotoID in this case, as PhotoID will itself be unique throughout the system.

How can we generate PhotoIDs? Here, we cannot have an auto-incrementing sequence in each shard to define PhotoID because we need to know PhotoID first to find the shard where it will be stored. One solution could be that we dedicate a separate database instance to generate auto-incrementing IDs. If our PhotoID can fit into 64 bits, we can define a table containing only a 64 bit ID field. So whenever we would like to add a photo in our system, we can insert a new row in this table and take that ID to be our PhotoID of the new photo.

Wouldn’t this key generating DB be a single point of failure? Yes, it would be. A workaround for that could be to define two such databases, one generating even-numbered IDs and the other odd-numbered.

We can put a load balancer in front of both of these databases to round-robin between them and to deal with downtime. Both these servers could be out of sync, with one generating more keys than the other, but this will not cause any issue in our system. We can extend this design by defining separate ID tables for Users, Photo-Comments, or other objects present in our system.

How can we plan for the future growth of our system? We can have a large number of logical partitions to accommodate future data growth, such that in the beginning, multiple logical partitions reside on a single physical database server. Since each database server can have multiple database instances running on it, we can have separate databases for each logical partition on any server. So whenever we feel that a particular database server has a lot of data, we can migrate some logical partitions from it to another server. We can maintain a config file (or a separate database) that can map our logical partitions to database servers; this will enable us to move partitions around easily. Whenever we want to move a partition, we only have to update the config file to announce the change.

<b>Ranking and News Feed Generation</b><br>
To create the News Feed for any given user, we need to fetch the latest, most popular, and relevant photos of the people the user follows.

For simplicity, let’s assume we need to fetch the top 100 photos for a user’s News Feed. Our application server will first get a list of people the user follows and then fetch metadata info of each user’s latest 100 photos. In the final step, the server will submit all these photos to our ranking algorithm, which will determine the top 100 photos (based on recency, likeness, etc.) and return them to the user. A possible problem with this approach would be higher latency as we have to query multiple tables and perform sorting/merging/ranking on the results. To improve the efficiency, we can pre-generate the News Feed and store it in a separate table.

Pre-generating the News Feed: We can have dedicated servers that are continuously generating users’ News Feeds and storing them in a ‘UserNewsFeed’ table. So whenever any user needs the latest photos for their News-Feed, we will simply query this table and return the results to the user.

Whenever these servers need to generate the News Feed of a user, they will first query the UserNewsFeed table to find the last time the News Feed was generated for that user. Then, new News-Feed data will be generated from that time onwards (following the steps mentioned above).

What are the different approaches for sending News Feed contents to the users?

1. Pull: Clients can pull the News-Feed contents from the server at a regular interval or manually whenever they need it. Possible problems with this approach are a) New data might not be shown to the users until clients issue a pull request b) Most of the time, pull requests will result in an empty response if there is no new data.

2. Push: Servers can push new data to the users as soon as it is available. To efficiently manage this, users have to maintain a Long Poll request with the server for receiving the updates. A possible problem with this approach is a user who follows a lot of people or a celebrity user who has millions of followers; in this case, the server has to push updates quite frequently.

3. Hybrid: We can adopt a hybrid approach. We can move all the users who have a high number of followers to a pull-based model and only push data to those who have a few hundred (or thousand) follows. Another approach could be that the server pushes updates to all the users not more than a certain frequency and letting users with a lot of updates to pull data regularly.

<b>Cache and Load balancing</b><br>
Our service would need a massive-scale photo delivery system to serve globally distributed users. Our service should push its content closer to the user using a large number of geographically distributed photo cache servers and use CDNs (for details, see Caching).

We can introduce a cache for metadata servers to cache hot database rows. We can use Memcache to cache the data.  Application servers can quickly check if the cache has desired rows before hitting the database. Least Recently Used (LRU) can be a reasonable cache eviction policy for our system. Under this policy, we discard the least recently viewed row first.

How can we build a more intelligent cache? If we go with the eighty-twenty rule, i.e., 20% of daily read volume for photos is generating 80% of the traffic, which means that certain photos are so popular that most people read them. This dictates that we can try caching 20% of the daily read volume of photos and metadata.
