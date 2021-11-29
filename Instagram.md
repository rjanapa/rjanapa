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

Step 3: Draw Logical Architecture: Block diagram for each Microservice, Data/Logic flow between them.

Step 4: Deep dive into each Microservice

Step 4a: For each Microservice – Data Model, How data is stored in Storage and Cache Tier, API, Workflow/Algorithm for API, Flow across Tiers

Step 4b: For each Microservice – check whether each tier needs to scale for storage, cache, throughput (CPU/IO), API parallelization, remove hotspots, Availability and Geo-Distribution

Step 4c: Draw a generic distributed architecture per tier
