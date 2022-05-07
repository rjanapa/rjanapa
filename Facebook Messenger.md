<b>Facebook Messenger</b>

Facebook Messenger allow users to send text messages to each other through web and mobile interfaces.

Step 1: Functional Requirements, Non-Functional Requirements, Extended Requirements, Design Constraints

<b>Functional Requirements:</b>

Messenger should support one-on-one conversations between users.<br>
Messenger should keep track of the online/offline statuses of its users.<br>
Messenger should support the persistent storage of chat history.

<b>Non-functional Requirements:</b>

Users should have a real-time chatting experience with minimum latency.<br>
Our system should be highly consistent; users should see the same chat history on all their devices.<br>

<b>Extended Requirements:</b>

Group Chats<br>
Push notifications: Messenger should be able to notify users of new messages when they are offline.

<b>Capacity Estimation and Constraints</b>
Assume 500 million daily active users, and on average, each user sends 40 messages daily; this gives us 20 billion messages per day.

<b>Storage Estimation:</b> Assume that, on average, a message is 100 bytes. So to store all the messages for one day, we would need 2TB of storage.

<b>20 billion messages * 100 bytes => 2000 GB/day => 2 TB/day</b><br>

<b>To store five years of chat history, we would need 3.6 petabytes of storage.</b><br>

2 TB * 365 days * 5 years ~= 4000 TB ~= 3.6 PB<br>

1 million bytes = 1 MB<br>
1000 MB or 1 billion bytes = 1 GB<br>
1000 GB = 1 TB<br>
1000 TB = 1 PB

Besides chat messages, we also need to store users’ information, messages’ metadata (ID, Timestamp, etc.). Not to mention, the above calculation doesn’t take data compression and replication into consideration.

<b>Bandwidth Estimation:</b> If our service is getting 2TB of data every day, this will give us 25MB of incoming data for each second.

2 TB / 86400 sec ~= 25 MB/s<br>

Since each incoming message needs to go out to another user, we will need the same amount of bandwidth 25MB/s for both upload and download.

Step 2: Define Microservice

Step 3: Draw Logical Architecture: Block diagram for each Microservice, Data/Logic flow between them.

<b>High Level Design</b><br>

At a high level, the system handle the following use cases:<br>

1. Receive incoming messages and deliver outgoing messages.<br>

2. Store and retrieve messages from the database.<br>

3. Keep a record of which user is online or has gone offline, and notify all the relevant users about these status changes.<br>

<img src="https://github.com/rjanapa/rjanapa/blob/main/FB-messenger-diagram.png" width="500" length="500">

<b>Design Summary:</b><br> 

Clients will open a connection to the chat server to send a message; the server will then pass it to the requested user. 

All the active users will keep a connection open with the server to receive messages. 

Whenever a new message arrives, the chat server will push it to the receiving user on the long poll request. 

Messages can be stored in HBase, which supports quick small updates and range-based searches. 

The servers can broadcast the online status of a user to other relevant users. Clients can pull status updates for users who are visible in the client’s viewport on a less frequent basis.

<b>Messages Handling</b><br>

To send messages, a user needs to connect to the server and post messages for the other users. To get a message from the server, the user has two options:

Pull model: Users can periodically ask the server if there are any new messages for them.<br>
Push model: Users can keep a connection open with the server and can depend upon the server to notify them whenever there are new messages.
In the first approach, the server needs to keep track of messages that are still waiting to be delivered, and as soon as the receiving user connects to the server to ask for any new message, the server can return all the pending messages. To minimize latency for the user, they have to check the server quite frequently, and most of the time, they will be getting an empty response if there are no pending messages. This will waste a lot of resources and does not look like an efficient solution.

If we go with our second approach, where all the active users keep a connection open with the server, then as soon as the server receives a message, it can immediately pass the message to the intended user. This way, the server does not need to keep track of the pending messages, and we will have minimum latency, as the messages are delivered instantly on the opened connection.

<b>Storage system to use</b><br>

We need to have a database that can support a very high rate of small updates and also fetch a range of records quickly. This is required because we have a huge number of small messages that need to be inserted in the database and, while querying, a user is mostly interested in sequentially accessing the messages.

We cannot use RDBMS like MySQL or NoSQL like MongoDB because we cannot afford to read/write a row from the database every time a user receives/sends a message. This will not only make the basic operations of our service run with high latency but also create a huge load on databases.

Both of our requirements can be easily met with a wide-column database solution like HBase. HBase is a column-oriented key-value NoSQL database that can store multiple values against one key into multiple columns. HBase is modeled after Google’s BigTable and runs on top of Hadoop Distributed File System (HDFS). HBase groups data together to store new data in a memory buffer and, once the buffer is full, it dumps the data to the disk. This way of storage not only helps to store a lot of small data quickly but also fetching rows by the key or scanning ranges of rows. HBase is also an efficient database to store variable-sized data, which is also required by our service.

<b>Data partitioning</b><br>

<b>Partitioning based on UserID: </b><br>
<b>Let’s assume we partition based on the hash of the UserID so that we can keep all messages of a user on the same database. If one DB shard is 4TB, we will have “3.6PB/4TB ~= 900” shards for five years. For simplicity, let’s assume we keep 1K shards. So we will find the shard number by “hash(UserID) % 1000” and then store/retrieve the data from there. This partitioning scheme will also be very quick to fetch chat history for any user.

Partitioning based on MessageID:</b><br>
If we store different messages of a user on separate database shards, fetching a range of messages of a chat would be very slow, so we should not adopt this scheme.

Step 4: Deep dive into each Microservice

Step 4a: For each Microservice – Data Model, How data is stored in Storage and Cache Tier, API, Workflow/Algorithm for API, Flow across Tiers

Step 4b: For each Microservice – check whether each tier needs to scale for storage, cache, throughput (CPU/IO), API parallelization, remove hotspots, Availability and Geo-Distribution

Step 4c: Draw a generic distributed architecture per tier
