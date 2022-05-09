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

