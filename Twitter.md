Twitter is an online social networking service where users post and read short 140-character messages called “tweets.

Registered users can post and read tweets, but those who are not registered can only read them. 

Users access Twitter through their website interface, SMS, or mobile app.

<b>Step 1: Functional Requirements, Non-Functional Requirements, Extended Requirements, Design Constraints</b><br>

<b>Functional Requirements</b><br>

Users should be able to post new tweets.<br>
A user should be able to follow other users.<br>
Users should be able to mark tweets as favorites.<br>
The service should be able to create and display a user’s timeline consisting of top tweets from all the people the user follows.<br>
Tweets can contain photos and videos.<br>

<b>Non-functional Requirements</b><br>

Our service needs to be highly available.<br>
Acceptable latency of the system is 200ms for timeline generation.<br>
Consistency can take a hit in the interest of availability; if a user doesn’t see a tweet for a while, it should be fine.<br>

Step 2: Define Microservice

Step 3: Draw Logical Architecture: Block diagram for each Microservice, Data/Logic flow between them.

Step 4: Deep dive into each Microservice

Step 4a: For each Microservice – Data Model, How data is stored in Storage and Cache Tier, API, Workflow/Algorithm for API, Flow across Tiers

Step 4b: For each Microservice – check whether each tier needs to scale for storage, cache, throughput (CPU/IO), API parallelization, remove hotspots, Availability and Geo-Distribution

Step 4c: Draw a generic distributed architecture per tier



