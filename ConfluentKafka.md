1) Team Alpha uses a MySQL database to store each rider's information. The database holds information such as rider's name, contact number, payment method, and more. 
Team Alpha can easily connect their database with Confluent Cloud by leveraging a fully-managed connector. 
The connector captures ongoing changes in the database and instantly replicate them to Confluent Cloud.
Kafka Connect provides integration for Kafka with other systems, both sending to and receiving data from them.

2) Storing data in Kafka topics
The source connector publishes messages to the rider_profiles topic. Now, this data is available to any team at Velocity that might need it!
A Kafka topic is a log where messages are sent to, stored, and retrieved from.

3) Syncing relational database with Confluent
Team Charlie handles each driver's information. PostgreSQL is the relational database of choice. 
This database stores information such as each driver's name, rating, and vehicle information. 
Team Charlie, uses a fully-managed source connector to connect their backend database with Confluent Cloud to keep them in sync with any changes.

4) Publishing data with Kafka producers
Team Bravo manages the mobile app installed on drivers' phones. Every second, the app produces the location of the driver and a riderID if there is a rider in the car at the time. 
As more drivers log into the app, the number of producer shown here increases.

5) Processing millions of data points
The real-time location for each vehicle is streamed to a Kafka topic named vehicle_locations. 
Team Bravo can use the data on available drivers on the road to initiate surge pricing or offer discounts to riders.

<img width="773" height="702" alt="image" src="https://github.com/user-attachments/assets/3a9f9264-0e70-4d11-bf84-8acb3d3cb5d6" />

6) Processing and enrichment of data in real time
Team Delta is responsible for creating and updating the fleet map that shows the live location of each vehicle. 
The map is continuously refreshed to show available drivers closest to each rider, as well as the drivers’ estimated time of arrival, rating, and price.

7) Getting an enriched stream
Team Delta consumes multiple data streams owned by other teams to create a fleet map. 
By using Confluent's stream processor Flink, Team Charlie is able to combine data from the driver_profiles, rider_profiles, and vehicle_locations topics 
to create a new data stream called driver_rider_vehicle_location. This data stream is then used to create and update the fleet map!
Flink is a way of processing streaming data in Kafka using SQL - Confluent offers the industry's only serverless, fully managed Flink service.

8) Transforming data and filtering
Team Echo is interested in creating a ride history for users by grouping location data into discrete rides. Each ride has a starting and stopping point, trip duration, and price. 
The team can easily write a ksqlDB query to process these data points and generate a rides stream. This stream can help the team offer promotions or launch new services for their users.
---------------------------------------------------

Kafka messages in .NET - Message<T Key, T Value>

Key is often string and can be some Id like UserId, OrderId etc. The data type of the key becomes important when you want to consider how you want to distribute your messages in order and impacts ordering guarantees in a cluster 

```
var message = new Message<String, Value>{
Key = biometrics.DeviceId,
Value = biometrics, // serialized as JSON/Avro/Protobuf. This object could be a domain entity/event that downstream systems would be interested in
//Optional
Headers = headers, //used to define the serializer used
Timestamp = new Timestamp(DateTime.UtcNow)
};

```
Be careful of using the metadata Headers and Timestamp - for instance overwriting Timestamp (populated by default) if needed by downstream systems.
Instead put them in the Value

