<b>Instagram</b><br>
Photo-sharing service that let users to upload photos to share with other users<br>
Instagram is a social networking service that enables its users to upload and share their photos and videos with other users. <br>
Instagram users can choose to share information either publicly or privately. <br>
Anything shared publicly can be seen by any other user, whereas privately shared content can only be accessed by the specified set of people. <br>
Instagram also enables its users to share through many other social networking platforms such as Facebook <br>
Design a simpler version of Instagram for this design problem, where a user can share photos and follow other users. <br>

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

The system would be read-heavy, so system should retrieve photos quickly.
Users can upload as many photos as they like; therefore, efficient management of storage is a crucial factor.<br>
Low latency is expected while viewing photos.<br>
Data should be 100% reliable. If a user uploads a photo, the system should guarantee that it will never be lost.<br>

<b>Step 2: Define Microservice</b><br>

Upload Image Microservice

View Image Microservice -> Download Image Microservice

Search Image Microservice -> Download Image Microservice

<b>Step 3: Draw Logical Architecture: Block diagram for each Microservice, Data/Logic flow between them.</b><br>

<b>High Level System Design Diagram</b><br>

<img src="https://github.com/rjanapa/rjanapa/blob/main/Instagram-high-level-systemdesign.png" width="500" length="500">

Photo uploads can be slow as they have to go to the disk, whereas reads will be faster, especially if they are being served from cache.

Uploading users can consume all the available connections, as uploading is a slow process. This means that ‘reads’ cannot be served if the system gets busy with all the ‘write’ requests. Web servers have a connection limit before designing our system. Assume that a web server have a maximum of 500 connections at any time, then it can’t have more than 500 concurrent uploads or reads. To handle this bottleneck, split reads and writes into separate services and have dedicated servers for reads and writes to ensure uploads don’t hog the system.

Separating photos’ read and write requests allow to scale and optimize each of these operations independently.

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


