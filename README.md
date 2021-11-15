- 👋 Hi, I’m @rjanapa
- 👀 I’m interested in Scalable System Design
- 🌱 I’m currently learning Scalable System Design
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me rjanapa@gmail.com

<!---
rjanapa/rjanapa is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->

URL Shortener

Step 1: 
Functional Req<b>
Given a long URL generate a short URL<b>
Given a short URL return long URL<b>
Generate custom URL<b>
TTL of Generated URL<b>
Analytics<b>

Design Constraints<b>
Number of URL generated per second<b>
Number of URL retrieved per second<b>
Size of Short URL. Assume 7 to start with.<b>
Characters in Short URL 0..9,a..z,A..Z<b>

Step 2: Define Microservices<b>

Step 3: Draw Logical Architecture<b><b>
Block Diagram for each Microservice<b>
Data/Logic flow between them<b>
Rules of Thumb: <b>
1. If high volume of data needs to be pushed in near real time between two Microservices, use Pub-Sub. Pub-Sub is a Microservice of its own.<b>
2. If data needs to be pulled from Server to Client, use REST API.<b>
3. If data transfer is offline use Batch ETL (Extract Transform Load) Job.<b>

Step 4: Deep dive into each Microservice at a time<b>
Each Microservice consists of one or more tiers<b>
App Server Tier - Application Logic<b>
Cache Server Tier - For high througput data access and in-memory compute<b>
Storage Server Tier - Data Persistence<b>

<img src="https://github.com/rjanapa/rjanapa/blob/main/3-tier-arch-diagram.png" width="250"><b>
