---
description: onContainerStart Functionality
---

# OnContainerStart

## What is it?

OnContainerStartFunction and on\_container\_start are functions you can optionally implement in any container to get execution, per operation, when the container starts up. This is helpful when your container needs to do some housekeeping and prep an agent, c2 profile, or even eventing before anything else happens.&#x20;

{% tabs %}
{% tab title="Python" %}
```python
class ContainerOnStartMessage:
    def __init__(self,
                 container_name: str = "",
                 operation_id: int = 0,
                 server_name: str = "",
                 apitoken: str = "",
                 **kwargs):
        self.ContainerName = container_name
        self.OperationID = operation_id
        self.ServerName = server_name
        self.APIToken = apitoken

    def to_json(self):
        return {
            "container_name": self.ContainerName,
            "operation_id": self.OperationID,
            "server_name": self.ServerName,
            "apitoken": self.APIToken
        }


class ContainerOnStartMessageResponse:
    def __init__(self,
                 ContainerName: str = "",
                 EventLogInfoMessage: str = "",
                 EventLogErrorMessage: str = ""):
        self.ContainerName = ContainerName
        self.EventLogInfoMessage = EventLogInfoMessage
        self.EventLogErrorMessage = EventLogErrorMessage

    def to_json(self):
        return {
            "container_name": self.ContainerName,
            "stdout": self.EventLogInfoMessage,
            "stderr": self.EventLogErrorMessage
        }
async def on_container_start(self, message: ContainerOnStartMessage) -> ContainerOnStartMessageResponse:
        return ContainerOnStartMessageResponse(ContainerName=self.name)
```
{% endtab %}

{% tab title="Golang" %}
```go
type ContainerOnStartMessage struct {
	ContainerName string `json:"container_name"`
	OperationID   int    `json:"operation_id"`
	OperationName string `json:"operation_name"`
	ServerName    string `json:"server_name"`
	APIToken      string `json:"apitoken"`
}

type ContainerOnStartMessageResponse struct {
	ContainerName        string `json:"container_name"`
	EventLogInfoMessage  string `json:"stdout"`
	EventLogErrorMessage string `json:"stderr"`
}
OnContainerStartFunction func(sharedStructs.ContainerOnStartMessage) sharedStructs.ContainerOnStartMessageResponse `json:"-"`
```
{% endtab %}
{% endtabs %}

## Where is it?

This function is one you can implement as part of the definition for your container (PayloadType, C2Profile, Eventing, etc).&#x20;

## What does it do?

This function gets an APIToken that is valid for 5 minutes and has the permissions of a spectator. This allows your container to query everything it needs, but not make any modifications.
