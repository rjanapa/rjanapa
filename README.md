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
Functional Req<p>
Given a long URL generate a short URL
Given a short URL return long URL
Generate custom URL
TTL of Generated URL
Analytics

Design Constraints
Number of URL generated per second
Number of URL retrieved per second
Size of Short URL. Assume 7 to start with.
Characters in Short URL 0..9,a..z,A..Z

Step 2: Define Microservices

Step 3: Draw Logical Architecture
Block Diagram for each Microservice
Data/Logic flow between them
Rules of Thumb: 
1. If high volume of data needs to be pushed in near real time between two Microservices, use Pub-Sub. Pub-Sub is a Microservice of its own.
2. If data needs to be pulled from Server to Client, use REST API.
3. If data transfer is offline use Batch ETL (Extract Transform Load) Job.

Step 4: Deep dive into each Microservice at a time
Each Microservice consists of one or more tiers
App Server Tier - Application Logic
Cache Server Tier - For high througput data access and in-memory compute
Storage Server Tier - Data Persistence

<img src="https://github.com/rjanapa/rjanapa/blob/main/3-tier-arch-diagram.png" width="250">
