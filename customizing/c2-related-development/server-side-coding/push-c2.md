# Push C2

## What is it?

Push C2 is a way to do egress C2 channels without requiring the agent to beacon periodically for tasking. Instead, a connection is held open and Mythic "Pushes" tasks and data down to the agent. This is very similar to how many Peer-to-peer (P2P) agents handle connections, just extended to the egress side of things.&#x20;

## How does it work?

For this to work, there needs to be a held open connection between the C2 Profile docker container and the Mythic server itself. This is done via gRPC. As part of this held open connection, the C2 profile identifies itself and forwards along messages.&#x20;

## What does it look like?

In the mythic UI, the last checkin time will change to 1970-01-01 and appear as `Streaming Now`. The moment the held open gRPC connection disconnects, that time will update to the current UTC time. This makes it very easy to know that a connection is currently held open even if no traffic is going through it.

Below are some simplified examples of working with this gRPC. An example of this Push style C2 as part of websockets is available with the `websocket` C2 profile.

{% tabs %}
{% tab title="First Tab" %}

{% endtab %}

{% tab title="Golang" %}
```go
PushConn := mythicGRPC.GetNewPushC2ClientConnection()
grpcClient := services.NewPushC2Client(PushConn)
streamContext, cancel := context.WithCancel(context.Background())
defer func() {
    cancel()
}()
grpcStream, err := grpcClient.StartPushC2Streaming(streamContext)
if err != nil {
	log.Printf("Failed to get new client: %v\n", err)
	return
} else {
	log.Printf("Got new push client")
}
// sending a message from an agent to Mythic
readErr = grpcStream.Send(&services.PushC2MessageFromAgent{
	C2ProfileName: "websocket",
	RemoteIP:      websocketClient.RemoteAddr().String(),
	TaskingSize:   0,
	Message:       nil,
	Base64Message: []byte(fromAgent.Data),
})
if readErr != nil {
	log.Printf("failed to send message to grpc stream: %v\n", readErr)
	grpcStream.CloseSend()
	return
}
// getting a message from Mythic
fromMythic, readErr := grpcStream.Recv()
if readErr != nil {
	log.Printf("Failed to read from grpc stream, closing connections: %v\n", readErr)
	grpcStream.CloseSend()
	return
}
dataContent := fromMythic.GetMessage()
```
{% endtab %}
{% endtabs %}
