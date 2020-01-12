# Serverless Fanout

Proof of concept of a distributed fanout messaging system that's completely serverless and uses minimal code. Practical
applications of this project include updating hot updating configuration on a group of clients, or busting on-client caches.

Distribute messages to currently connected subscribers with built in retries and an acknowledgement mechanism. Subscribers
connect via websockets to an API Gateway endpoint, connection management and message distribution are handled by Step Functions.  

At-least-once delivery semantics.

# Infrastructure

- Dynamo DB
- API Gateway
- Step Functions

# Message Formats

Incoming messages are wrapped and distributed over currently open sockets, using the following JSON message format: 

    {
        "id": <unique message id>
        "payload": <base64 encoded payload>
    }
    
Subscribers process the message and respond with an acknowledgement message containing the unique id
of the original message over the websocket.

    {
        "ack": <processed message id>
    }
    
If an acknowledgement message is not received within the configured timeout, the message will be resent a configurable
amount of times.

# Subscriber Management

Active websocket connections are tracked within a Dynamo table. Incoming messages will be distributed to all currently
connected clients.

Stale websocket connections will be reaped based on a configurable time period. 

# Configuration

- Wait for acknowledgement timeout
- Message resend times
- Stale websocket connection timeout
