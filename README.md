- 👋 Hi, I’m @rjanapa
- 👀 I’m interested in Scalable System Design
- 🌱 I’m currently learning Scalable System Design
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me rjanapa@gmail.com

<!---
rjanapa/rjanapa is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->

<b>URL Shortener</b><br>

Step 1:<br> 
Functional Req<br>
Given a long URL generate a short URL<br>
Given a short URL return long URL<br>
Generate custom URL<br>
TTL of Generated URL<br>
Analytics<br>

Design Constraints<br>
Number of URL generated per second<br>
Number of URL retrieved per second<br>
Size of Short URL. Assume 7 to start with.<br>
Characters in Short URL 0..9,a..z,A..Z<br>

Step 2: Define Microservices<br>

Step 3: Draw Logical Architecture<br>
Block Diagram for each Microservice<br>
Data/Logic flow between them<br>
Rules of Thumb: <br>
1. If high volume of data needs to be pushed in near real time between two Microservices, use Pub-Sub. Pub-Sub is a Microservice of its own.<br>
2. If data needs to be pulled from Server to Client, use REST API.<br>
3. If data transfer is offline use Batch ETL (Extract Transform Load) Job.<br>

Step 4: Deep dive into each Microservice at a time<br>
Each Microservice consists of one or more tiers<br>
App Server Tier - Application Logic<br>
Cache Server Tier - For high througput data access and in-memory compute<br>
Storage Server Tier - Data Persistence<br>

<img src="https://github.com/rjanapa/rjanapa/blob/main/3-tier-arch-diagram.png" width="250"><br>

Data Model<br>
Short URL/Unique Id, Long URL, TTL, Creation Time<br>
k: Short URL/Unique Id<br>
v: Long URL, TTL, Creation Time<br>

API<br> Similar to CRUD operations<br>
create(Long URL) -> create(v)<br><br><br><br><br><br>
read(Short URL) -> read(k)<br><br><br><br><br>
update(k,v)<br><br><br><br>
delete(k)<br><br><br>

How to organize data<br><br>
HashMap<br>









