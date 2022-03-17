<b>System Design Facebook Messenger</b>

Instant messaging service like Facebook Messenger allow users to send text messages to each other through web and mobile interfaces.

Facebook Messenger is a software application that provides text-based instant messaging services to its users. Messenger users can chat with their Facebook friends both from cell phones and Facebook’s website.

Step 1: Functional Requirements, Non-Functional Requirements, Extended Requirements, Design Constraints

<b>Functional Requirements:</b>

Messenger should support one-on-one conversations between users.<br>
Messenger should keep track of the online/offline statuses of its users.<br>
Messenger should support the persistent storage of chat history.

<b>Non-functional Requirements:</b>

Users should have a real-time chatting experience with minimum latency.<br>
Our system should be highly consistent; users should see the same chat history on all their devices.<br>
Messenger’s high availability is desirable; we can tolerate lower availability in the interest of consistency.

<b>Extended Requirements:</b>

Group Chats: Messenger should support multiple people talking to each other in a group.<br>
Push notifications: Messenger should be able to notify users of new messages when they are offline.

<b>Capacity Estimation and Constraints</b>
Assume 500 million daily active users, and on average, each user sends 40 messages daily; this gives us 20 billion messages per day.

<b>Storage Estimation:</b> Assume that, on average, a message is 100 bytes. So to store all the messages for one day, we would need 2TB of storage.

20 billion messages * 100 bytes => 2 TB/day<br><br>
To store five years of chat history, we would need 3.6 petabytes of storage.<br>

2 TB * 365 days * 5 years ~= 3.6 PB<br>
Besides chat messages, we also need to store users’ information, messages’ metadata (ID, Timestamp, etc.). Not to mention, the above calculation doesn’t take data compression and replication into consideration.

<b>Bandwidth Estimation:</b> If our service is getting 2TB of data every day, this will give us 25MB of incoming data for each second.

2 TB / 86400 sec ~= 25 MB/s<br>
Since each incoming message needs to go out to another user, we will need the same amount of bandwidth 25MB/s for both upload and download.

Step 2: Define Microservice

Step 3: Draw Logical Architecture: Block diagram for each Microservice, Data/Logic flow between them.

Step 4: Deep dive into each Microservice

Step 4a: For each Microservice – Data Model, How data is stored in Storage and Cache Tier, API, Workflow/Algorithm for API, Flow across Tiers

Step 4b: For each Microservice – check whether each tier needs to scale for storage, cache, throughput (CPU/IO), API parallelization, remove hotspots, Availability and Geo-Distribution

Step 4c: Draw a generic distributed architecture per tier
