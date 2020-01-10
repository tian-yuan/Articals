#### Receive offline mqtt message



In order to have your client as a durable client and receive messages that were sent to topics when it was offline, you need to meet the following criteria:

* Fixed client ID (as you've done)
* Always connect with clean_session=False
* Subscriptions must be made with QoS>0
* Messages published must have QoS>0

The mistake that I make most frequently is to forget either one of points 3 and 4, so I'm publishing with QoS=0 or subscribing with QoS=0, either of which would cause messages not to be stored.

You could also look at the `queue_qos0_messages` option to tell the broker to store QoS=0 messages as well. Note that this is an implementation detail that may be specific to mosquitto

