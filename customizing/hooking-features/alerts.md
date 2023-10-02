# Alerts

## Agent generated alerts

Sometimes you want your agent to be able to bring something to the operator's attention. Your task might return an error, but if it's a long-running task, then who knows if the operator is actively looking at the task.&#x20;

### alerts

Using the `alerts` keyword, agents can sent alert messages to Mythic's operation event log.  There are two ways you can do this:

#### As part of a task

As part of a task, you can use another keyword, `alerts` to send the following structure:\


```
"responses": [
    {
        "task_id": "some uuid here",
        "alerts": [
            {
                "source": "source of this message", //optional
                "alert": "the alert message you want to send to mythic"
            }
        ]
]

```

You can send multiple alerts at once since it's an array.

{% hint style="warning" %}
The `source` field doesn't get displayed to the user, but it is used to collapse like-messages within the UI. If the user has an alert that's not resolved with a source of "bob", then the next message with a source of "bob" will **NOT** be displayed, but will instead simply increase the count of the current message. If the user resolves the message with a source of "bob" and a new one comes in, then that message **WILL** be displayed. This is to help prevent users from getting flooded with messages.


{% endhint %}

{% hint style="info" %}
The "squishing" of alert messages only happens in the UI - if you have a webhook container and are listening for alerts, you will get _all_ of the messages. The `basic_webhook` container has code in it to throttle alert messages of the same source though that they must be a minute apart (again, to help prevent spam to users)
{% endhint %}

#### Not as part of a task

Sometimes you have other aspects to your agent that might be monitoring or running tasks and you want to report that data back to the user, but you don't have a specific task to associate it with. In that case, you can do the exact same `alerts` structure, but at a higher level in the structure:

```
{
    "action": "get_tasking", // any action
    "alerts": [
            {
                "source": "source of this message", //optional
                "alert": "the alert message you want to send to mythic"
            }
        ]
}
```



## Extended Format

The base format is an `alert` to display in the UI and the `source` of the alert. This can be extended though to provide more customization:

```
{
    "action": "some action",
    "alerts": [
        {
            "source": "String source of message", // optional
            "level": "String level: warning, info, debug", // optional
            "send_webhook": true, // optional boolean to send a webhook regardless of if this is a warning or not
            "alert": "String message to display in UI event feed", // optional
            "webhook_alert": {}, // optional dictionary data for webhook
        }
    ]
}
```

This extended format has a few more fields:

* `send_webhook` - normally, if the `level` is not provided or is `warning` then an `alert` webhook is sent. However, if you set the level to something like `info` or `debug`, then you can optionally still force a webhook with this flag
* `level` - this identifies how `alert` is presented to the user. `warning` is the default and will show a warning error as well as put a warning notification in the event log (increasing the warning count in the UI by 1). `info` is similar - it displays an informative message in the UI to the user and adds a message to the event log, but it isn't a warning message that needs to be addressed. `debug` will allow you to send a message to the event log, but it will _not_ display a toast notification to the user.
* `alert` - your normal string message that's displayed to the user and put in the event log
* `webhook_alert` - optional dictionary data that doesn't get displayed to the user or put in the operation event log, but is instead sent along as custom data to the `custom_webhook` webhook for additional processing. Specifically, the data sent to the `custom_webhook` is as follows:

```go
if err := RabbitMQConnection.EmitWebhookMessage(WebhookMessage{
	OperationID:      operationInfo.ID,
	OperationName:    operationInfo.Name,
	OperationWebhook: operationInfo.Webhook,
	OperationChannel: operationInfo.Channel,
	OperatorUsername: "",
	Action:           WEBHOOK_TYPE_CUSTOM,
	Data: map[string]interface{}{
		"callback_id":   callbackID,
		"alert":         alert.Alert,
		"webhook_alert": alert.WebhookAlert,
		"source":        alert.Source,
	},
}); err != nil {
	logging.LogError(err, "Failed to send webhook")
}
```
