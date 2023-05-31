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
                "source": "source of this message",
                "alert": "the alert message you want to send to mythic"
            }
        ]
]

```

You can send multiple alerts at once since it's an array.

{% hint style="warning" %}
The `source` field doesn't get displayed to the user, but it is used to collapse like-messages within the UI. If the user has an alert that's not resolved with a source of "bob", then the next message with a source of "bob" will **NOT** be displayed, but will instead simply increase the count of the current message. If the user resolves the message with a source of "bob" and a new one comes in, then that message **WILL** be displayed. This is to help prevent users from getting flooded with messages.
{% endhint %}

#### Not as part of a task

Sometimes you have other aspects to your agent that might be monitoring or running tasks and you want to report that data back to the user, but you don't have a specific task to associate it with. In that case, you can do the exact same `alerts` structure, but at a higher level in the structure:

```
{
    "action": "get_tasking", // any action
    "alerts": [
            {
                "source": "source of this message",
                "alert": "the alert message you want to send to mythic"
            }
        ]
}
```
