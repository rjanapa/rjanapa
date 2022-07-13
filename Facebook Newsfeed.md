<b>Facebook’s Newsfeed</b><br>

Facebook's Newsfeed contains posts, photos, videos, and status updates from all the people and pages a user follows.

<b>Requirements</b><br>

<b>Functional requirements:</b><br>

1. Newsfeed will be generated based on the posts from the people, pages, and groups that a user follows.<br>
2. A user may have many friends and follow a large number of pages/groups.<br>
3. Feeds may contain images, videos, or just text.<br>
4. The service should support appending new posts as they arrive to the newsfeed for all active users.<br>

<b>Non-functional requirements:</b><br>

1. The system should be able to generate any user’s newsfeed in real-time. Maximum latency seen by the end user would be 2s.<br>
2. A post shouldn’t take more than 5s to make it to a user’s feed assuming a new newsfeed request comes in.<br>

<b>System APIs:</b><br>

getUserFeed(api_dev_key, user_id, since_id, count, max_id, exclude_replies)<br>

Parameters:<br>
api_dev_key (string): The API developer key of a registered can be used to, among other things, throttle users based on their allocated quota.<br>
user_id (number): The ID of the user for whom the system will generate the newsfeed.<br>

<b>Database Design:</b><br>

There are three primary objects: User, Entity (e.g. page, group, etc.), and FeedItem (or Post). Here are some observations about the relationships between these entities:

A User can follow other entities and can become friends with other users.<br>
Both users and entities can post FeedItems which can contain text, images, or videos.<br>
Each FeedItem will have a UserID which will point to the User who created it. For simplicity, let’s assume that only users can create feed items, although, on Facebook Pages can post feed item too.<br>
Each FeedItem can optionally have an EntityID pointing to the page or the group where that post was created.<br>

<img src="https://github.com/rjanapa/rjanapa/blob/main/FacebookNewsFeedDBDesign.png" width="500" length="500">

<b>High Level System Design</b><br>

At a high level this system can be divided into two parts:

<b>Feed generation:</b><br>

Newsfeed is generated from the posts (or feed items) of users and entities (pages and groups) that a user follows. 

Whenever the system receives a request to generate the feed for a user, the following steps are performed:

1. Retrieve IDs of all users and entities that the user follows.<br>
2. Retrieve latest, most popular and relevant posts for those IDs. These are the potential posts that we can show in user’s newsfeed.<br>
3. Rank these posts based on the relevance to user. This represents user’s current feed.<br>
4. Store this feed in the cache and return top posts (say 20) to be rendered on user’s feed.<br>
5. On the front-end, when user reaches the end of the current feed, fetch the next 20 posts from the server and so on.<br>

<b>Components in Newsfeed service: </b><br>

<b>Web servers: </b> To maintain a connection with the user. This connection will be used to transfer data between the user and the server.<br>

<b>Application server:</b> To execute the workflows of storing new posts in the database servers. We will also need some application servers to retrieve and to push the newsfeed to the end user.<br>

<b>Metadata database and cache:</b> To store the metadata about Users, Pages, and Groups.<br>

<b>Posts database and cache:</b> To store metadata about posts and their contents.<br>

<b>Video and photo storage, and cache:</b> Blob storage, to store all the media included in the posts.<br>

<b>Newsfeed generation service:</b> To gather and rank all the relevant posts for a user to generate newsfeed and store in the cache. This service will also receive live updates and will add these newer feed items to any user’s timeline.<br>

<b>Feed notification service:</b> To notify the user that there are newer items available for their newsfeed.<br>

<img src="https://github.com/rjanapa/rjanapa/blob/main/FB-Newsfeed-HLD.png" width="500" length="500">

<b>Detailed Component Design</b><br>

<b>a. Feed generation</b><br>

Newsfeed generation service fetching most recent posts from all the users and entities that the user follows<br>

(SELECT FeedItemID FROM FeedItem WHERE UserID in (<br>
    SELECT EntityOrFriendID FROM UserFollow WHERE UserID = <current_user_id> and type = 0(user))<br>
)<br>
UNION<br>
(SELECT FeedItemID FROM FeedItem WHERE EntityID in (<br>
    SELECT EntityOrFriendID FROM UserFollow WHERE UserID = <current_user_id> and type = 1(entity))<br>
)<br>
ORDER BY CreationDate DESC <br>
LIMIT 100<br>

To improve the efficiency, pre-generate the timeline and store it in a memory.<br>

<b>Offline generation for newsfeed: </b><br>

Dedicated servers can continuously generating users’ newsfeed and storing them in memory. So, whenever a user requests for the new posts for their feed, we can simply serve it from the pre-generated, stored location. Using this scheme, user’s newsfeed is not compiled on load, but rather on a regular basis and returned to users whenever they request for it.<br>

Whenever these servers need to generate the feed for a user, they will first query to see what was the last time the feed was generated for that user. Then, new feed data would be generated from that time onwards. We can store this data in a hash table where the “key” would be UserID and “value” would be a STRUCT like this:<br>

Struct {<br>
    LinkedHashMap<FeedItemID, FeedItem> feedItems;<br>
    DateTime lastGenerated;<br>
}<br>

Use an LRU based cache that can remove users from memory that haven’t accessed their newsfeed for a long time 

<b>b. Feed publishing</b><br>

<b>"Pull" model: </b><br>

This method involves keeping all the recent feed data in memory so that users can pull it from the server whenever they need it. 

It’s hard to find the right pull cadence, as most of the time pull requests will result in an empty response if there is no new data, causing waste of resources.

<b>"Push" model:</b><br>

For a push system, once a user has published a post, we can immediately push this post to all the followers. A possible problem with this approach is that when a user has millions of followers (a celebrity-user) the server has to push updates to a lot of people.

<b>Hybrid:</b><br>

An alternate method to handle feed data could be to use a hybrid approach, Specifically, we can stop pushing posts from users with a high number of followers (a celebrity user) and only push data for those users who have a few hundred (or thousand) followers. For celebrity users, we can let the followers pull the updates. Since the push operation can be extremely costly for users who have a lot of friends or followers





