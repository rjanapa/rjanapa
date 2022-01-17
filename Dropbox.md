<b>Dropbox</b><br>

<b>System Design</b><br>

<b>Step 1: Functional Requirements, Non-Functional Requirements, Extended Requirements, Design Constraints</b><br>

<b>File hosting service</b><br>
Cloud file storage enables users to store their data on remote servers. <br>
The servers are maintained by cloud storage providers and made available to users over a network typically through the Internet. <br>
Users pay for their cloud data storage on a monthly basis.<br>

<b>Why Cloud Storage?</b><br>

The shift from using personal computers to using multiple devices with different platforms and operating systems such as smartphones and tablets each with portable access from various geographical locations at any time, is accountable for the huge popularity of cloud storage services. 

Following are some of the top benefits of such services:

Availability: The motto of cloud storage services is to have data availability anywhere, anytime. Users can access their files/photos from any device whenever and wherever they like.

Reliability and Durability: It offers 100% reliability and durability of data. Cloud storage ensures that users will never lose their data by keeping multiple copies of the data stored on different geographically located servers.

Scalability: Users will never have to worry about running out of storage space. With cloud storage one get unlimited storage as long as one is ready to pay for it.

<b>Requirements and Goals of the System</b><br>

Here are the top-level requirements for our system:<br>

1. Users should be able to upload and download their files/photos from any device.<br>
2. Users should be able to share files or folders with other users.<br>
3. Our service should support automatic synchronization between devices, i.e., after updating a file on one device, it should get synchronized on all devices.<br>
4. The system should support storing large files up to a GB.<br>
5. ACID-ity is required. Atomicity, Consistency, Isolation and Durability of all file operations should be guaranteed.<br> https://database.guide/what-is-acid-in-databases/ <br>
6. Our system should support offline editing. Users should be able to add/delete/modify files while offline, and as soon as they come online, all their changes should be synced to the remote servers and other online devices.<br>

<b>Extended Requirements</b><br>

The system should support snapshotting of the data, so that users can go back to any version of the files.

<b>Some Design Considerations</b><br>

1. We should expect huge read and write volumes.
2. Read to write ratio is expected to be nearly the same.
3. Internally, files can be stored in small parts or chunks (say 4MB); this can provide a lot of benefits i.e. all failed operations shall only be retried for smaller parts of a file. If a user fails to upload a file, then only the failing chunk will be retried.
4. We can reduce the amount of data exchange by transferring updated chunks only.
5. By removing duplicate chunks, we can save storage space and bandwidth usage.
6. Keeping a local copy of the metadata (file name, size, etc.) with the client can save us a lot of round trips to the server.
7. For small changes, clients can intelligently upload the diffs instead of the whole chunk.

<b>Capacity Estimation and Constraints</b><br>
Assume total users = 100M, and daily active users (DAU) = 10M.<br>
Assume on average each user connects from three different devices.<br>
On average if a user has 100 files/photos, we will have 1 billion total files.<br>
Assume that average file size is 100KB, this would give us ten petabytes of total storage.<br>
1B * 100KB => 100TB<br>
Assume that we will have one million active connections per minute.<br>

<b>High Level Design</b><br>

The user will specify a folder as the workspace on their device. Any file/photo/folder placed in this folder will be uploaded to the cloud, and whenever a file is modified or deleted, it will be reflected in the same way in the cloud storage. The user can specify similar workspaces on all their devices and any modification done on one device will be propagated to all other devices to have the same view of the workspace everywhere.

At a high level, we need to store files and their metadata information like File Name, File Size, Directory, etc., and who this file is shared with. 

<b>Block Servers</b>: Servers that help the clients to upload/download files to Cloud Storage

<b>Metadata Servers</b>: Servers that facilitate updating metadata about files and users. Metadata servers will keep metadata of files updated in a SQL or NoSQL database

<b>Synchronization Servers</b>: Mechanism to notify all clients whenever an update happens so they can synchronize their files. Synchronization servers handle the workflow of notifying all clients about different changes for synchronization.

<b>Component Design</b><br>

The major components of the system are:

<b>a. Client</b><br>

The Client Application monitors the workspace folder on the user’s machine and syncs all files/folders in it with the remote Cloud Storage. The client application will work with the storage servers to upload, download, and modify actual files to backend Cloud Storage. The client also interacts with the remote Synchronization Service to handle any file metadata updates, e.g., change in the file name, size, modification date, etc.

Some of the essential operations for the client:

1) Upload and download files.
2) Detect file changes in the workspace folder.
3) Handle conflict due to offline or concurrent updates.

How is file transfer handled efficiently. Each file is broken into smaller chunks so that only those chunks that are modified are transfered and not the whole file. Suppose divide each file into fixed sizes of 4MB chunks. One can statically calculate optimal chunk size based on 

1) Storage devices used in the cloud to optimize space utilization and input/output operations per second (IOPS) 
2) Network bandwidth 
3) Average file size in the storage etc. 
In metadata, keep a record of each file and the chunks that constitute it.

Do one keep a copy of metadata with Clients? 
Keeping a local copy of metadata enables offline updates but also saves a lot of round trips to update remote metadata.

How can clients efficiently listen to changes happening with other clients? 
One solution could be clients periodically check with the server if there are any changes. The problem with this approach is that delay in reflecting changes locally as clients will be checking for changes periodically compared to a server notifying whenever there is some change. If the client frequently checks the server for changes, it will not only be wasting bandwidth, as the server has to return an empty response most of the time, but will also be keeping the server busy. Pulling information in this manner is not scalable.

A solution to the above problem could be to use HTTP long polling. With long polling, the client requests information from the server with the expectation that the server may not respond immediately. If the server has no new data for the client when the poll is received, instead of sending an empty response, the server holds the request open and waits for response information to become available. Once it does have new information, the server immediately sends an HTTP/S response to the client, completing the open HTTP/S Request. Upon receipt of the server response, the client can immediately issue another server request for future updates.

Based on the above considerations, one can divide our client into four parts:

1) <b>Internal Metadata Database:</b> Keep track of all the files, chunks, their versions, and their location in the file system.

2) <b>Chunker:</b> Split the files into smaller pieces called chunks. Also responsible for reconstructing a file from its chunks. The chunking algorithm will detect the parts of the files that have been modified by the user and only transfer those parts to the Cloud Storage; this will save bandwidth and synchronization time.

3) <b>Watcher:</b> Monitor the local workspace folders and notify the Indexer (discussed below) of any action performed by the users, e.g. when users create, delete, or update files or folders. Watcher also listens to any changes happening on other clients that are broadcasted by Synchronization service.

4) <b>Indexer:</b> will process the events received from the Watcher and update the internal metadata database with information about the chunks of the modified files. Once the chunks are successfully submitted/downloaded to the Cloud Storage, the Indexer will communicate with the remote Synchronization Service to broadcast changes to other clients and update the remote metadata database.

<b>b. Metadata Database</b>
The Metadata Database is responsible for maintaining the versioning and metadata information about files/chunks, users, and workspaces. The Metadata Database can be a relational database such as MySQL or a NoSQL database service such as DynamoDB. Regardless of the type of the database, the Synchronization Service should be able to provide a consistent view of the files using a database, especially if more than one user is working with the same file simultaneously. Since NoSQL data stores do not support ACID properties in favor of scalability and performance, one need to incorporate the support for ACID properties programmatically in the logic of our Synchronization Service in case one opt for this kind of database. However, using a relational database can simplify the implementation of the Synchronization Service as they natively support ACID properties.

The metadata Database should be storing information about following objects:

1) Chunks
2) Files
3) User
4) Devices
5) Workspace (sync folders)

<b>c. Synchronization Service</b>
The Synchronization Service is the component that processes file updates made by a client and applies these changes to other subscribed clients. It also synchronizes clients’ local databases with the information stored in the remote Metadata DB. The Synchronization Service is the most important part of the system architecture due to its critical role in managing the metadata and synchronizing users’ files. Desktop clients communicate with the Synchronization Service to either obtain updates from the Cloud Storage or send files and updates to the Cloud Storage and, potentially, other users. If a client was offline for a period, it polls the system for new updates as soon as they come online. When the Synchronization Service receives an update request, it checks with the Metadata Database for consistency and then proceeds with the update. Subsequently, a notification is sent to all subscribed users or devices to report the file update.

The Synchronization Service should be designed to transmit less data between clients and the Cloud Storage to achieve a better response time. To meet this design goal, the Synchronization Service can employ a differencing algorithm to reduce the amount of data that needs to be synchronized. Instead of transmitting entire files from clients to the server or vice versa, one can just transmit the difference between two versions of a file. Therefore, only the part of the file that has been changed is transmitted. This also decreases bandwidth consumption and cloud data storage for the end-user. One will be dividing files into 4MB chunks and will be transferring modified chunks only. Server and clients can calculate a hash (e.g., SHA-256) to see whether to update the local copy of a chunk or not. On the server, if one already have a chunk with a similar hash (even from another user), one don’t need to create another copy; one can use the same chunk.  

To be able to provide an efficient and scalable synchronization protocol, one can consider using a communication middleware between clients and the Synchronization Service. The messaging middleware should provide scalable message queuing and change notifications to support a high number of clients using pull or push strategies. This way, multiple Synchronization Service instances can receive requests from a global request Queue, and the communication middleware will be able to balance its load.

<b>d. Message Queuing Service</b>
An important part of our architecture is a messaging middleware that should be able to handle a substantial number of requests. A scalable Message Queuing Service that supports asynchronous message-based communication between clients and the Synchronization Service best fits the requirements of our application. The Message Queuing Service supports asynchronous and loosely coupled message-based communication between distributed components of the system. The Message Queuing Service should be able to efficiently store any number of messages in a highly available, reliable, and scalable queue.

The Message Queuing Service will implement two types of queues in our system. The Request Queue is a global queue and all clients will share it. Clients’ requests to update the Metadata Database will be sent to the Request Queue first; from there, the Synchronization Service will take it to update metadata. The Response Queues that correspond to individual subscribed clients are responsible for delivering the update messages to each client. Since a message will be deleted from the queue once received by a client, we need to create separate Response Queues for each subscribed client to share update messages.

<b>e. Cloud/Block Storage</b>
Cloud/Block Storage stores chunks of files uploaded by the users. Clients directly interact with the storage to send and receive objects from it. Separation of the metadata from storage enables us to use any storage either in the cloud or in-house.

Step 2: Define Microservice

Step 3: Draw Logical Architecture: Block diagram for each Microservice, Data/Logic flow between them.

Step 4: Deep dive into each Microservice

Step 4a: For each Microservice – Data Model, How data is stored in Storage and Cache Tier, API, Workflow/Algorithm for API, Flow across Tiers

Step 4b: For each Microservice – check whether each tier needs to scale for storage, cache, throughput (CPU/IO), API parallelization, remove hotspots, Availability and Geo-Distribution

Step 4c: Draw a generic distributed architecture per tier