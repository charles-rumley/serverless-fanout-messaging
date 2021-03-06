# Serverless Fanout

Proof of concept of a distributed fanout messaging system that's completely serverless and uses minimal code. Practical
applications of this project include updating hot updates of configuration on a group of clients, or busting on-client caches.

Distribute messages to currently connected subscribers with built in retries and an acknowledgement mechanism. Subscribers
connect via websockets to an API Gateway endpoint, connection management and message distribution are handled by Step Functions.  

At-least-once delivery semantics.

## Infrastructure

- Dynamo DB
- API Gateway
- Step Functions

## Message Formats

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

## Subscriber Management

Active websocket connections are tracked within a Dynamo table. Incoming messages will be distributed to all currently
connected clients.

Stale websocket connections will be reaped based on a configurable time period. 

## Configuration

- Wait for acknowledgement timeout
- Message resend times
- Stale websocket connection timeout

## Testing

The infrastructure spinup is handled completely by CloudFormation. Launch the stack, then grab the API Gateway websocket
connection endpoint from the CloudFormation stack output.

### Create a client

    npm install -g wscat

    # simulate multiple clients by running this command in several
    # different terminal sessions
    wscat -c wss://<endpoint details>.amazonaws.com/test

TODO Explain the `test` portion of the URL

### Send a message

    TODO Invoke a Lambda function with the CLI
