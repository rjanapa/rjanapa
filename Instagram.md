<b>Instagram</b><br>

Instagram is a social networking service that enables its users to upload and share their photos and videos with other users. <br>

A user can follow other users. <br
                                   
The ‘News Feed’ for each user will consist of top photos of all the people the user follows<br>

<b>Step 1: </b>

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

<b>Step 3: High Level System Design Diagram</b><br>

<img src="https://github.com/rjanapa/rjanapa/blob/main/Instagram-high-level-diagram.png" width="500" length="500">

<b>Component Design</b><br>

Photo uploads can be slow as they have to go to the disk, whereas reads will be faster, especially if they are being served from cache.

Split reads and writes into separate services and have dedicated servers for reads and writes to ensure uploads don’t hog the system.

Separating photos’ read and write requests allow to scale and optimize each of these operations independently.

<b>Reliability and Redundancy</b><br>

If one wants to have high availability of the system, one need to have multiple replicas of services running in the system so that even if a few services die down, the system remains available and running. Redundancy removes the single point of failure in the system.<br>

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

One approach to store the above schema would be to use an RDBMS like MySQL since it require joins. But RDBMS come with their own challenges, especially when one needs to scale them. <br>

Store photos in a distributed file storage like HDFS (Hadoop Distributed File System) or S3.<br>

Store the above schema in a distributed key-value store to enjoy the benefits offered by NoSQL. All the metadata related to photos go to a table where the ‘key’ is‘PhotoID’ and the ‘value’ is an object containing (PhotoLocation, UserLocation, CreationTimestamp, etc).

If one go with a NoSQL database, need an additional table  ‘UserPhoto’ to store the relationships between users and photos to know who owns which photo.  <br>

Also need a table ‘UserFollow’ to store the list of people a user follows. <br>

For both the tables, Use a wide-column datastore like Cassandra. For the ‘UserPhoto’ table, the ‘key’ would be ‘UserID’, and the ‘value’ would be the list of ‘PhotoIDs’ the user owns, stored in different columns. Have a similar scheme for the ‘UserFollow’ table.

<b>Data Sharding</b><br>

<b>a. Partitioning based on UserID </b><br>

Shard based on the ‘UserID’ so that keep all photos of a user on the same shard.   

Different issues with this approach:<br>
Several people follow hot users e.g. celebrities, and a lot of other people see any photo they upload.<br>
Some users will have a lot of photos compared to others, thus making a non-uniform distribution of storage.<br>
Storing all photos of a user on one shard can cause issues like unavailability of all of the user’s data if that shard is down or higher latency if it is serving high load etc.<br>

<b>b. Partitioning based on PhotoID </b><br>

If one can generate unique PhotoIDs first and then find a shard number through “PhotoID % 10”, the above problems will be solved.

How can we generate PhotoIDs? <br>
Here, we cannot have an auto-incrementing sequence in each shard to define PhotoID because we need to know PhotoID first to find the shard where it will be stored. One solution could be that we dedicate a separate database instance to generate auto-incrementing IDs. 

Wouldn’t this key generating DB be a single point of failure? <br>
Yes, it would be. A workaround for that could be to define two such databases, one generating even-numbered IDs and the other odd-numbered.

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

