<b>Dropbox</b><br>

File Hosting Service.<br>

Cloud file storage enables users to store their data on remote servers.<br>

The servers are maintained by cloud storage providers and made available to users over a network typically through the Internet. <br>

Users pay for their cloud data storage on a monthly basis.<br>

<b>System Design</b><br>

<b>Step 1: Functional Requirements, Non-Functional Requirements, Extended Requirements, Design Constraints</b><br>

<b>Functional Requirements:</b><br>

1. Users should be able to upload and download their files/photos from any device.<br>
2. Users should be able to share files or folders with other users.<br>
3. The service should support automatic synchronization between devices, i.e., after updating a file on one device, it should get synchronized on all devices.<br>
4. The system should support storing large files up to a GB.<br>
5. ACID-ity is required. Atomicity, Consistency, Isolation and Durability of all file operations should be guaranteed.<br> https://github.com/rjanapa/rjanapa/edit/main/ACID.md
6. The system should support offline editing. Users should be able to add/delete/modify files while offline, and as soon as they come online, all their changes should be synced to the remote servers and other online devices.<br>

<b>Non-Functional Requirements:</b>

<b>Availability:</b> Data availability anywhere, anytime. Users can access their files/photos from any device whenever and wherever they like.

<b>Reliability and Durability:</b> It offers 100% reliability and durability of data. Cloud storage ensures that users never lose their data by storing multiple copies of the data on different geographically distributed servers.

<b>Scalability:</b> Users don't have to worry about running out of storage space. With cloud storage one get unlimited storage as long as one is ready to pay for it.

<b>Extended Requirements</b><br>

The system should support snapshotting of the data, so that users can go back to any version of the files.

<b>Some Design Considerations</b><br>

1. High read and write volumes.
2. Read to write ratio is nearly same.

<b>Capacity Estimation and Constraints</b><br>
Assume total users = 100M, and daily active users (DAU) = 10M.<br>
On average if a user has 10 files/photos, we will have 1 billion total files.<br>
Assume that average file size is 100KB, this would give us 100 GB of total storage.<br>
1B * 100KB => 100 GB<br>
Assume that we will have one million active connections per minute.<br>

<b>System Design</b><br>

<b>High Level Design</b><br>
<img src="https://github.com/rjanapa/rjanapa/blob/main/Dropbox-HLD.png" width="500" length="500">

<b>File Processing Workflow</b>
The sequence below shows the interaction between the components of the application in a scenario when Client A updates a file that is shared with Client B and C, so they should receive the update too. If the other clients are not online at the time of the update, the Message Queuing Service keeps the update notifications in separate response queues for them until they come online later.

1) Client A uploads chunks to cloud storage.
2) Client A updates metadata and commits changes.
3) Client A gets confirmation and notifications are sent to Clients B and C about the changes.
4) Client B and C receive metadata changes and download updated chunks.

<b>Architecture Diagram</b><br>
<img src="https://github.com/rjanapa/rjanapa/blob/main/DropboxDesign.png" width="500" length="500">

<b>Component Design</b><br>

The major components of the system are:

<b>1. Client</b><br>

One can divide client app into following parts:

<b>Workspace</b>: The user specify a folder as the workspace on their device. Any file/photo/folder placed in this folder will be uploaded to the cloud, and whenever a file is modified or deleted, it will be reflected in the same way in the cloud storage. The client application monitors the workspace folder on the user’s machine and syncs all files/folders in it with the remote Cloud Storage. The client application work with the storage servers to upload, download, and modify actual files to backend cloud storage. The client also interacts with the remote Synchronization Service to handle any file metadata updates, e.g., change in the file name, size, modification date, etc.

<b>Chunker:</b> Split the files into smaller pieces called chunks. Also responsible for reconstructing a file from its chunks. The chunking algorithm will detect the parts of the files that have been modified by the user and only transfer those parts to the Cloud Storage; this will save bandwidth and synchronization time. That is how file transfer is handled efficiently. Each file is broken into smaller chunks so that only those chunks that are modified are transfered and not the whole file. Internally, files can be stored in small parts or chunks (say 4MB). his provides a lot of benefits i.e. all failed operations shall only be retried for smaller parts of a file. If a user fails to upload a file, then only the failing chunk will be retried. We can reduce the amount of data exchange by transferring updated chunks only. By removing duplicate chunks, we can save storage space and bandwidth usage. For small changes, clients can intelligently upload the diffs instead of the whole chunk.

<b>Internal Metadata Database:</b> Keep track of all the files, chunks, their versions and location in the file system. Keeping a local copy of the metadata (file name, size, etc.) with the client save  a lot of round trips to the server.


<b>Watcher:</b> Monitor the local workspace folders and notify the Indexer of any action performed by the users, e.g. when users create, delete, or update files or folders. Watcher also listens to any changes happening on other clients that are broadcasted by Synchronization service.

<b>Indexer:</b> will process the events received from the Watcher and update the internal metadata database with information about the chunks of the modified files. Once the chunks are successfully submitted/downloaded to the Cloud Storage, the Indexer will communicate with the remote Synchronization Service to broadcast changes to other clients and update the remote metadata database.

Some of the essential operations for the client: a) Upload and download files. b) Detect file changes in the workspace folder. c) Handle conflict due to offline or concurrent updates.

How can clients efficiently listen to changes happening with other clients? 
One solution could be clients periodically check with the server if there are any changes. The problem with this approach is that delay in reflecting changes locally as clients will be checking for changes periodically compared to a server notifying whenever there is some change. If the client frequently checks the server for changes, it will not only be wasting bandwidth, as the server has to return an empty response most of the time, but will also be keeping the server busy. Pulling information in this manner is not scalable.

A solution to the above problem could be to use HTTP long polling. With long polling, the client requests information from the server with the expectation that the server may not respond immediately. If the server has no new data for the client when the poll is received, instead of sending an empty response, the server holds the request open and waits for response information to become available. Once it does have new information, the server immediately sends an HTTP/S response to the client, completing the open HTTP/S Request. Upon receipt of the server response, the client can immediately issue another server request for future updates.

<b>2. Block Servers</b>: Servers that help the clients to upload/download files to Cloud Block Storage

<b>3. Cloud Block Storage</b>
Cloud Block Storage stores chunks of files uploaded by the users. Clients directly interact with the storage to send and receive objects from it. Separation of the metadata from storage enables us to use any storage either in the cloud or in-house.

<b>4. Metadata Servers</b>: Servers that facilitate updating metadata about files and users. Metadata servers will keep metadata of files updated in a SQL or NoSQL database

<b>5. Metadata DB:</b> At a high level, one needs to store files and their metadata information like File Name, File Size, Directory, etc., and who this file is shared with. 
<ins>The Metadata Database is responsible for maintaining the versioning and metadata information about files/chunks, users, and workspaces.</ins> The Metadata Database can be a relational database such as MySQL or a NoSQL database service such as DynamoDB. Regardless of the type of the database, the Synchronization Service should be able to provide a consistent view of the files using a database, especially if more than one user is working with the same file simultaneously. Since NoSQL data stores do not support ACID properties in favor of scalability and performance, one need to incorporate the support for ACID properties programmatically in the logic of our Synchronization Service in case one opt for this kind of database. However, using a relational database can simplify the implementation of the Synchronization Service as they natively support ACID properties.
<ins>The metadata Database should be storing information about following objects: Chunks, Files, User, Devices, Workspace (sync folders)</ins>.

<b>Metadata Partitioning</b>
To scale out metadata DB, we need to partition it so that it can store information about millions of users and billions of files/chunks. We need to come up with a partitioning scheme that would divide and store our data in different DB servers.

1. Vertical Partitioning: We can partition our database in such a way that we store tables related to one particular feature on one server. For example, we can store all the user-related tables in one database and all files/chunks related tables in another database. Although this approach is straightforward to implement it has some issues:
Joining two tables in two separate databases can cause performance and consistency issues.<br>

2. Range Based Partitioning: What if we store files/chunks in separate partitions based on the first letter of the File Path? In that case, we save all the files starting with the letter ‘A’ in one partition and those that start with the letter ‘B’ into another partition and so on. This approach is called range-based partitioning. The main problem with this approach is that it can lead to unbalanced servers. For example, if we decide to put all files starting with the letter ‘E’ into a DB partition, and later we realize that we have too many files that start with the letter ‘E’, to such an extent that we cannot fit them into one DB partition.

3. Hash-Based Partitioning: We take a hash of the object that we are storing and based on this hash we figure out the DB partition to which this object should go. In our case, we can take the hash of the ‘FileID’ of the file object we are storing to determine the partition the file will be stored. The hashing function will randomly distribute objects into different partitions, e.g., hashing function can always map any ID to a number between [1…256], and this number would be the partition we will store our object.

<b>6. Synchronization Service:</b> Synchronization mechanism to notify all clients whenever an update happens so they can synchronize their files. Synchronization Servers handle the workflow of notifying all clients about different changes for synchronization. <ins>The Synchronization Service is the component that processes file updates made by a client and applies these changes to other subscribed clients. It also synchronizes clients’ local databases with the information stored in the Metadata DB. The Synchronization Service is the most important part of the system architecture due to its critical role in managing the metadata and synchronizing users’ files.</ins> Desktop clients communicate with the Synchronization Service to either obtain updates from the Cloud Storage or send files and updates to the Cloud Storage and, potentially, other users. If a client was offline for a period, it polls the system for new updates as soon as they come online. When the Synchronization Service receives an update request, it checks with the Metadata DB for consistency and then proceeds with the update. Subsequently, a notification is sent to all subscribed users or devices to report the file update.

The Synchronization Service should be designed to transmit less data between clients and the Cloud Block Storage to achieve a better response time. To meet this design goal, the Synchronization Service can employ a differencing algorithm to reduce the amount of data that needs to be synchronized. Instead of transmitting entire files from clients to the server or vice versa, one can just transmit the difference between two versions of a file. Therefore, only the part of the file that has been changed is transmitted. This also decreases bandwidth consumption and cloud data storage for the end-user. One will be dividing files into 4MB chunks and will be transferring modified chunks only. Server and clients can calculate a hash (e.g., SHA-256) to see whether to update the local copy of a chunk or not. On the server, if one already have a chunk with a similar hash (even from another user), one don’t need to create another copy; one can use the same chunk.  

To be able to provide an efficient and scalable synchronization protocol, one can consider using a communication middleware between clients and the Synchronization Service. The messaging middleware should provide scalable message queuing and change notifications to support a high number of clients using pull or push strategies. This way, multiple Synchronization Service instances can receive requests from a global request Queue, and the communication middleware will be able to balance its load.

<b>7. Message Queuing Service</b>
An important part of our architecture is a messaging middleware that should be able to handle a substantial number of requests. <ins>A scalable Message Queuing Service that supports asynchronous message-based communication between clients and the Synchronization Service best fits the requirements of our application. The Message Queuing Service supports asynchronous and loosely coupled message-based communication between distributed components of the system. The Message Queuing Service should be able to efficiently store any number of messages in a highly available, reliable, and scalable queue.</ins> 

The Message Queuing Service will implement two types of queues in our system. The Request Queue is a global queue and all clients will share it. Clients’ requests to update the Metadata Database will be sent to the Request Queue first; from there, the Synchronization Service will take it to update metadata. The Response Queues that correspond to individual subscribed clients are responsible for delivering the update messages to each client. Since a message will be deleted from the queue once received by a client, we need to create separate Response Queues for each subscribed client to share update messages.

<b>Data Deduplication</b>
Data deduplication is a technique used for eliminating duplicate copies of data to improve storage utilization. It can also be applied to network data transfers to reduce the number of bytes that must be sent. For each new incoming chunk, we can calculate a hash of it and compare that hash with all the hashes of the existing chunks to see if we already have the same chunk present in our storage.

We can implement deduplication in two ways in our system:

a. Post-process deduplication
With post-process deduplication, new chunks are first stored on the storage device and later some process analyzes the data looking for duplication. The benefit is that clients will not need to wait for the hash calculation or lookup to complete before storing the data, thereby ensuring that there is no degradation in storage performance. Drawbacks of this approach are 1) We will unnecessarily be storing duplicate data, though for a short time, 2) Duplicate data will be transferred consuming bandwidth.

b. In-line deduplication
Alternatively, deduplication hash calculations can be done in real-time as the clients are entering data on their device. If our system identifies a chunk that it has already stored, only a reference to the existing chunk will be added in the metadata, rather than a full copy of the chunk. This approach will give us optimal network and storage usage.

<b>Caching</b>
We can have two kinds of caches in our system. To deal with hot files/chunks we can introduce a cache for block storage. We can use an off-the-shelf solution like Memcached that can store whole chunks with its respective IDs/Hashes and Block Servers before hitting Block Storage can quickly check if the cache has desired chunk. Based on clients’ usage patterns we can determine how many cache servers we need. A high-end commercial server can have 144GB of memory; one such server can cache 36K chunks.

Which cache replacement policy would best fit our needs? When the cache is full, and we want to replace a chunk with a newer/hotter chunk, how would we choose? Least Recently Used (LRU) can be a reasonable policy for our system. Under this policy, we discard the least recently used chunk first. 

Similarly, we can have a cache for Metadata DB.

<b>Load Balancer (LB)</b>
We can add the Load balancing layer at two places in our system: 1) Between Clients and Block servers and 2) Between Clients and Metadata servers. Initially, a simple Round Robin approach can be adopted that distributes incoming requests equally among backend servers. This LB is simple to implement and does not introduce any overhead. Another benefit of this approach is if a server is dead, LB will take it out of the rotation and will stop sending any traffic to it. A problem with Round Robin LB is, it won’t take server load into consideration. If a server is overloaded or slow, the LB will not stop sending new requests to that server. To handle this, a more intelligent LB solution can be placed that periodically queries backend servers about their load and adjusts traffic based on that.

<b>Security, Permissions and File Sharing</b>
One of the primary concerns users will have while storing their files in the cloud is the privacy and security of their data, especially since in our system users can share their files with other users or even make them public to share them with everyone. To handle this, we will be storing the permissions of each file in our metadata DB to reflect what files are visible or modifiable by any user.

Step 2: Define Microservice

Step 3: Draw Logical Architecture: Block diagram for each Microservice, Data/Logic flow between them.

Step 4: Deep dive into each Microservice

Step 4a: For each Microservice – Data Model, How data is stored in Storage and Cache Tier, API, Workflow/Algorithm for API, Flow across Tiers

Step 4b: For each Microservice – check whether each tier needs to scale for storage, cache, throughput (CPU/IO), API parallelization, remove hotspots, Availability and Geo-Distribution

Step 4c: Draw a generic distributed architecture per tier
