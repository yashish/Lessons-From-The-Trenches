"In essence a Domain event signals something that has happened in the outside world that is of interest to the application" - Martin Fowler

An event is something that has happened in the past. It's immutable and cannot be changed in the present. We can only act upon it - unless we can time travel back in the past!

1. Use past tense when naming events. Eg, User Created - and not Create User
2. Be descriptive about the event name. Eg, User Address Changed - and not User Updated
Once we have identified what an event is, package it into a message. For instance, in Kafka message<Key,Value> as KV pair

```csharp
var message = new Message<Key, Value>{  //string, Biometrics
  Key = "Some string",   //biometrics.DeviceId
  Value = "Some Value",   //biometrics
  Headers = headers,
  Timestamp = new Timestamp(DateTime.Now)
};
```

Normally Keys are some primitive string or int values. Determines how data is distributed in the Kafka cluster and impacts ordering of the messages.
Normally the Key will be an identifier of a domain entity, like a DeviceId or UserId.

The Value typically stores the details of the event. It could be a simple string message but generally it will be a complex object that can be serilaized as JSON, Avro or Protobuf. This might be a representation of the event or a domain entity that's relevant to the event.

The message also contains optional metadata like headers or timestamp. The Timestamp is populated by default but we can overwrite. The headers is important to provide information about the serilaization method used to help deserialize the Value. However, if knowing the timestamp is important for downstream applications, put it as part of the Value rather than hiding it in the metadata.

