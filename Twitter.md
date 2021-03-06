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

<b>System APIs</b><br>

tweet(api_dev_key, tweet_data, tweet_location, user_location, media_ids)

Parameters:<br>
api_dev_key (string): The API developer key of a registered account. This will be used to, among other things, throttle users based on their allocated quota.<br>
tweet_data (string): The text of the tweet, typically up to 140 characters.<br>
tweet_location (string): Optional location (longitude, latitude) this Tweet refers to.<br>
user_location (string): Optional location (longitude, latitude) of the user adding the tweet.<br>
media_ids (number[]): Optional list of media_ids to be associated with the Tweet. (all the media photo, video, etc. need to be uploaded separately).<br>

Returns: (string)
A successful post will return the URL to access that tweet. Otherwise, an appropriate HTTP error is returned.

Step 3: Draw Logical Architecture: Block diagram for each Microservice, Data/Logic flow between them.

<img src="https://github.com/rjanapa/rjanapa/blob/main/TwitterHLD.png" width="500" length="500">

<img src="https://github.com/rjanapa/rjanapa/blob/main/TwitterDBTables.png" width="500" length="500">

<b>Data Sharding</b><br>

<b>Sharding based on UserID:</b> Storing all the data of a user on one server. While storing, pass the UserID to our hash function that will map the user to a database server where store all of the user’s tweets, favorites, follows, etc. While querying for tweets/follows/favorites of a user, ask the hash function where can one find the data of a user and then read it from there. This approach has issue: What if a user becomes hot? There could be a lot of queries on the server holding the user. This high load will affect the performance of our service.

<b>Sharding based on TweetID:</b> The hash function will map each TweetID to a random server where we will store that Tweet. To search for tweets, we have to query all servers, and each server will return a set of tweets. A centralized server will aggregate these results to return them to the user. Let’s look into timeline generation example; here are the steps the system perform to generate a user’s timeline:

The application (app) server find all the people the user follows.
App server send the query to all database servers to find tweets from these people.
Each database server find the tweets for each user, sort them by recency and return the top tweets.
App server merge all the results and sort them again to return the top results to the user.
This approach solves the problem of hot users, but, in contrast to sharding by UserID, one have to query all database partitions to find tweets of a user, which can result in higher latencies.

We can further improve our performance by introducing cache to store hot tweets in front of the database servers.

Step 4: Deep dive into each Microservice

Step 4a: For each Microservice – Data Model, How data is stored in Storage and Cache Tier, API, Workflow/Algorithm for API, Flow across Tiers

Step 4b: For each Microservice – check whether each tier needs to scale for storage, cache, throughput (CPU/IO), API parallelization, remove hotspots, Availability and Geo-Distribution

Step 4c: Draw a generic distributed architecture per tier



