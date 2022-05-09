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

<b>High Level System Design</b><br>

At a high level this system can be divided into two parts:

<b>Feed generation:</b><br>

Newsfeed is generated from the posts (or feed items) of users and entities (pages and groups) that a user follows. So, whenever the system receives a request to generate the feed for a user, the following steps will be performed:

Retrieve IDs of all users and entities that the user follows.<br>
Retrieve latest, most popular and relevant posts for those IDs. These are the potential posts that we can show in user’s newsfeed.<br>
Rank these posts based on the relevance to user. This represents user’s current feed.<br>
Store this feed in the cache and return top posts (say 20) to be rendered on user’s feed.<br>
On the front-end, when user reaches the end of the current feed, fetch the next 20 posts from the server and so on.<br>
