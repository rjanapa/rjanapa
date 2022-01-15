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

<b>Some Design Considerations</b><br>

1. We should expect huge read and write volumes.
2. Read to write ratio is expected to be nearly the same.
3. Internally, files can be stored in small parts or chunks (say 4MB); this can provide a lot of benefits i.e. all failed operations shall only be retried for smaller parts of a file. If a user fails to upload a file, then only the failing chunk will be retried.
4. We can reduce the amount of data exchange by transferring updated chunks only.
5. By removing duplicate chunks, we can save storage space and bandwidth usage.
6. Keeping a local copy of the metadata (file name, size, etc.) with the client can save us a lot of round trips to the server.
7. For small changes, clients can intelligently upload the diffs instead of the whole chunk.

<b>Capacity Estimation and Constraints</b><br>
Assume total users = 100M, and daily active users (DAU) = 10M.<br>
Assume on average each user connects from three different devices.<br>
On average if a user has 100 files/photos, we will have 1 billion total files.<br>
Assume that average file size is 100KB, this would give us ten petabytes of total storage.<br>
1B * 100KB => 100TB<br>
Assume that we will have one million active connections per minute.<br>
