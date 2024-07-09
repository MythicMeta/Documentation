# Webhooks

## Webhook Structure

Webhooks are notified of certain events in an asynchronous manner and submit that data to pre-configured webhook URLs. Webhooks can take advantage of everything you can do via Scripting by using the MythicRPCAPITokenCreate functionality. This function provides a temporary, trackable API token that can be used to interact with the GraphQL API. The benefit here is that if you want or need more information than what's directly provided by the webhook message, you can fetch it from GraphQL.



{% tabs %}
{% tab title="Python" %}

{% endtab %}

{% tab title="Golang" %}
```go
type WebhookDefinition struct {
	Name                     string
	Description              string
	WebhookURL               string
	WebhookChannel           string
	NewFeedbackFunction      func(input NewFeedbackWebookMessage)
	NewCallbackFunction      func(input NewCallbackWebookMessage)
	NewStartupFunction       func(input NewStartupWebhookMessage)
	NewAlertFunction         func(input NewAlertWebhookMessage)
	NewCustomFunction        func(input NewCustomWebhookMessage)
	Subscriptions            []string
	OnContainerStartFunction func(sharedStructs.ContainerOnStartMessage) sharedStructs.ContainerOnStartMessageResponse
}
```

for example:

```go
func Initialize() {
	myWebhooks := webhookstructs.WebhookDefinition{
		Name:                "my_webhooks",
		Description:         "default webhook for slack example",
		NewFeedbackFunction: newfeedbackWebhook,
		NewCallbackFunction: newCallbackWebhook,
		NewStartupFunction:  newStartupMessage,
	}
	webhookstructs.AllWebhookData.Get("my_webhooks").AddWebhookDefinition(myWebhooks)
}
```

There's also a built-in function you can leverage to get the webhook url and channel:

```go
func newCallbackWebhook(input webhookstructs.NewCallbackWebookMessage) {
    newMessage := webhookstructs.GetNewDefaultWebhookMessage()
    newMessage.Channel = webhookstructs.AllWebhookData.Get("my_webhooks").GetWebhookChannel(input, webhookstructs.WEBHOOK_TYPE_NEW_CALLBACK)
    var webhookURL = webhookstructs.AllWebhookData.Get("my_webhooks").GetWebhookURL(input, webhookstructs.WEBHOOK_TYPE_NEW_CALLBACK)
    if webhookURL == "" {
       logging.LogError(nil, "No webhook url specified for operation or locally")
       go mythicrpc.SendMythicRPCOperationEventLogCreate(mythicrpc.MythicRPCOperationEventLogCreateMessage{
          OperationId:  &input.OperationID,
          Message:      "No webhook url specified, can't send webhook message",
          MessageLevel: mythicrpc.MESSAGE_LEVEL_WARNING,
       })
       return
    }
   ...
}
```
{% endtab %}
{% endtabs %}

* `Name` - this is the name of your webhook container
* `Description` - this is the description for your container (probably provides insight if you're going to submit to Slack, Discord, or some other service)
*   `WebhookURL` - this is an optional URL you can configure for the actual webhook to use. Configuring it here makes it take the highest precedence when it comes time to actually send the webhook, but has the downside of being hardcoded. You can also optionally configure this on a per-operation basis in your Operation in the UI. The last place you can configure this is in the .env file for Mythic as the&#x20;

    ```
    WEBHOOK_DEFAULT_URL
    ```

    variable.
* `WebhookChannel` - similar to the `WebhookURL`, this is the channel you're going to send your webhook. This can also be configured via the `.env` as a series of `WEBHOOK_DEFAULT_*_CHANNEL` to allow you to configure a different channel per type of notification.
* The `*Function`s are what get executed when an event of that type happens. If, for example, you don't want to handle processing NewFeedback messages from Mythic, then you can simply not provide a function here (or set it to `nil` / `None` explicitly) and Mythic won't even bother sending the notification down to your container.
* `OnContainerStartFunction` - this allows you to perform additional processing/setup when your container comes online and syncs up with Mythic. You get a temporary (5min) Spectator token **for each active operation**. This means that if there are two active operations in Mythic, then this function gets called **twice**, once for each operation. This is so that if you need to do some sort of configuration that's specific to an operation, you can fetch data for that operation.
