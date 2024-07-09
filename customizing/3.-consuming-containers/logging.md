# Logging

## Logging Structure

Many organizations that use Mythic have a need for the logs generated from it, either for tracking user actions, deconflictions, or as part of operations (such as purple teaming). Different teams also have different levels of detail needed from logs, different formats, and even different output styles (stdout, files, direct to a SIEM, etc). Because of this, Mythic outsources _most_ of its logs to logging containers that can subscribe to various events and then do whatever they want with the data. The nice thing about having this as part of a container that's hooked up to Mythic is that you can get the initial logging message and then turn around and use MythicRPC and Mythic's Scripting to augment that data with more context that you might need before shipping it off.



{% tabs %}
{% tab title="Python" %}

{% endtab %}

{% tab title="Golang" %}
```go
type LoggingDefinition struct {
	Name                     string
	Description              string
	LogToFilePath            string
	LogLevel                 string
	LogMaxSizeInMB           int
	LogMaxBackups            int
	NewCallbackFunction      func(input NewCallbackLog)
	NewCredentialFunction    func(input NewCredentialLog)
	NewKeylogFunction        func(input NewKeylogLog)
	NewFileFunction          func(input NewFileLog)
	NewPayloadFunction       func(input NewPayloadLog)
	NewArtifactFunction      func(input NewArtifactLog)
	NewTaskFunction          func(input NewTaskLog)
	NewResponseFunction      func(input NewResponseLog)
	Subscriptions            []string
	OnContainerStartFunction func(sharedStructs.ContainerOnStartMessage) sharedStructs.ContainerOnStartMessageResponse
}
```

and creating it:

```go
myLoggerName := "my_logger"
myLogger := loggingstructs.LoggingDefinition{
    NewCallbackFunction: func(input loggingstructs.NewCallbackLog) {
       loggingstructs.AllLoggingData.Get(myLoggerName).LogInfo(input.Action, "data", input)
    },
    NewTaskFunction: func(input loggingstructs.NewTaskLog) {
       loggingstructs.AllLoggingData.Get(myLoggerName).LogInfo(input.Action, "data", input.Data)
    },
    NewPayloadFunction: func(input loggingstructs.NewPayloadLog) {
       loggingstructs.AllLoggingData.Get(myLoggerName).LogInfo(input.Action, "data", input.Data)
    },
    NewKeylogFunction: func(input loggingstructs.NewKeylogLog) {
       loggingstructs.AllLoggingData.Get(myLoggerName).LogInfo(input.Action, "data", input.Data)
    },
    NewCredentialFunction: func(input loggingstructs.NewCredentialLog) {
       loggingstructs.AllLoggingData.Get(myLoggerName).LogInfo(input.Action, "data", input.Data)
    },
    NewArtifactFunction: func(input loggingstructs.NewArtifactLog) {
       loggingstructs.AllLoggingData.Get(myLoggerName).LogInfo(input.Action, "data", input.Data)
    },
    NewFileFunction: func(input loggingstructs.NewFileLog) {
       loggingstructs.AllLoggingData.Get(myLoggerName).LogInfo(input.Action, "data", input.Data)
    },
}
loggingstructs.AllLoggingData.Get(myLoggerName).AddLoggingDefinition(myLogger)
```

In this example we're just using the built-in logger and writing to stdout.
{% endtab %}
{% endtabs %}

Most of these fields in the definition are pretty self explanatory. You don't need to fill out `subscriptions` though - that is auto populated based on which functions you provide and is used to update the MythicUI to indicate what logs you're collecting. In the Go example above and screenshot below, we didn't register a function for new responses, so in the UI you can see that the "test" button for new responses is disabled.

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>
