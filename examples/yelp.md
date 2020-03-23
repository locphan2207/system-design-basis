# Design a service like Yelp

### Requirements:
1. Functional:
- User can search nearby restaurents based on categories and radius
- Owners can create/update/delete the restaurent info.
Q: _"Which of these features do you want me to focus on?"_
A: _"Search nearby restaurents"_

2. Non-functional:
_ Expect high availability
- Fast read

### Scale Estimation:
Q: _"How many users do you expect this system to handle? Do you expect any growth in the future?"_
A:
- Active users per day: 500M
- Searches per day: 100K
- Number of restaurents: 500M
- Growth: 20% number of places each year.

### Database Models and Storage:
1. Users:
- user_id: 8 bytes 
- long: 8 bytes
- lat: 8 bytes
-> 24 bytes
-> 12 GB for 500M users a day to store their locations

2. Locations:
- location_id: 8 bytes
- long: 8 bytes  
- lat: 8 bytes
- rating: 1 byte
- description: 256 bytes
-> 264 bytes
-> 132 GB for 500M restaurents

### API interface:
- GET: Queries:
  - session_token: 8 bytes  
  - user_id: 8 bytes
  - long: 8 bytes
  - lat: 8 bytes
  - radius: 1 byte
-> 33 bytes
-> 3.3 GB bandwidth for incoming traffic for 100K requests a day

### High level design:
Clients send request to application server. Application server query with that location to get back 50 closest restaurents.
- More than 1 app server to scale horizontally 
- Two load balancer (active-active) are in front of app servers. Weighted Round-robin should work.
- LRU Cache is key-value store with key is long-lat and value is list of restaurents. Can match close locations to a key that is a few feets away.
- 2 quad-tree servers (active-active). Meaning 2 LB will also be between app servers and quad-tree servers.
- Database follows master-slave. Master will do write and replicate data to slaves. 2 slaves take care of read since this system is read-heavy.

### Detail design:
1. QuadTree for faster search:
  - Each grid in the QuadTree can keep at most 500 restaurent ids 
  - QuadTree node can split into 4 childrens in case there are more than 500 restaurents in the area.
  - Each grid stores the range of long and lat. To split, just find the mid of each range.
  - Make sure only leaves nodes store the restaurent ids.
  - Each node store a reference of its parent. In case it needs to go back up to get more results.
  - QuadTree size: location_id (8 bytes) * 500M = 4 GB
- Basically, when the app server receives a request with a user current location. It will use it to traverse down the QuadTree to the leaf. In case the result is not enough, visit its siblings by going back to the parent node. Do it until it gather enough result (50 restaurents) or until it reachs the radius.

2. Partitioning and sharding:
- Could use NoSQL since this is read heavy and data structure won't change too much.
- We can shard data based on regions but that can result in unevenly work if a region is more popular.
- So instead, we can do hash from location id. Each shard can have a corresponding quad tree. However, since we have many quadtrees, we need to aggregate the results into one before returning the response.
