<b>Dropbox</b><br>

<b>File hosting service</b><br>
Cloud file storage enables users to store their data on remote servers. <br>
The servers are maintained by cloud storage providers and made available to users over a network typically through the Internet. <br>
Users pay for their cloud data storage on a monthly basis.<br>

<b>Why Cloud Storage?</b><br>

The shift from using personal computers to using multiple devices with different platforms and operating systems such as smartphones and tablets each with portable access from various geographical locations at any time, is accountable for the huge popularity of cloud storage services. 

Following are some of the top benefits of such services:

Availability: The motto of cloud storage services is to have data availability anywhere, anytime. Users can access their files/photos from any device whenever and wherever they like.

Reliability and Durability: It offers 100% reliability and durability of data. Cloud storage ensures that users will never lose their data by keeping multiple copies of the data stored on different geographically located servers.

Scalability: Users will never have to worry about running out of storage space. With cloud storage one get unlimited storage as long as one is ready to pay for it.

<b>Requirements and Goals of the System</b><br>

Here are the top-level requirements for our system:<br>

1. Users should be able to upload and download their files/photos from any device.<br>
2. Users should be able to share files or folders with other users.<br>
3. Our service should support automatic synchronization between devices, i.e., after updating a file on one device, it should get synchronized on all devices.<br>
4. The system should support storing large files up to a GB.<br>
5. ACID-ity is required. Atomicity, Consistency, Isolation and Durability of all file operations should be guaranteed.<br> https://database.guide/what-is-acid-in-databases/ <br>
6. Our system should support offline editing. Users should be able to add/delete/modify files while offline, and as soon as they come online, all their changes should be synced to the remote servers and other online devices.<br>

<b>Extended Requirements</b><br>

The system should support snapshotting of the data, so that users can go back to any version of the files.



