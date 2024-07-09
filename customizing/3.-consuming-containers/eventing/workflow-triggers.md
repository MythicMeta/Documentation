# Workflow Triggers

## What are they?

Triggers are the "events" that kick off your workflow. They typically involve something "happening" within Mythic's sphere of influence and sometimes allow you to add some additional context via `trigger_data`.&#x20;

{% hint style="info" %}
`trigger_data` isn't set in stone and can be expanded upon over time. If you have additional ideas for trigger data, let me know!
{% endhint %}

## Trigger Options

* `manual` - This workflow is triggered manually in the UI via the green run icon.
  * `trigger_data` - N/A
* `keyword` - This workflow is triggered by a keyword and optional dictionary of contextual data.
  * `trigger_data` - dictionary of any extra data you want to send along. Normally, this is an _extra_ way of triggering a workflow that's normally triggered in another way. In that case, you should probably pass along in the `trigger_data` whatever your workflow normally expects.
* `mythic_start` - This workflow is triggered when Mythic starts.
  * `trigger_data` - N/A
* `cron` - This workflow is triggered on a cron schedule.
  * `trigger_data` - Dictionary with the following keys:
    * `cron` - a normal `cron` string indicating when you want to execute this workflow. This is a handy place to check out for cron execution strings ([https://crontab.guru/](https://crontab.guru/)).&#x20;
* `payload_build_start` - This workflow is triggered when a Payload first starts being built.
  * `trigger_data` - Dictionary with the following keys:
    * `payload_types` - a list of all the payload types where you _want_ this to trigger. If you don't specify any, then it will trigger for _all_ payload types.
* `payload_build_finish` - This workflow is triggered when a Payload finishes being built (either successfully or with an error).
  * `trigger_data` - Dictionary with the following keys:
    * `payload_types` - a list of all the payload types where you _want_ this to trigger. If you don't specify any, then it will trigger for _all_ payload types.
* `task_create` - This workflow is triggered when a Task is first created and sent for preprocessing.
  * `trigger_data` - N/A
* `task_start` - This workflow is triggered when a Task is picked up by an agent to start executing.
  * `trigger_data` - N/A
* `task_finish` - This workflow is triggered when a Task finishes (at any point in the task lifecycle) either successfully or with an error.
  * `trigger_data` - N/A
* `user_output` - This workflow is triggered when a Task returns new output in the `user_output` field for the user to see in the UI.
  * `trigger_data` - N/A
* `file_download` - This workflow is triggered when a file finishes downloading from a callback.
  * `trigger_data` - N/A
* `file_upload` - This workflow is triggered when a file finishes uploading to Mythic.
  * `trigger_data` - N/A
* `screenshot` - This workflow is triggered when a screenshot finishes downloading from a callback.
  * `trigger_data` - N/A
* `alert` - This workflow is triggered when an agent sends an alert back to Mythic.&#x20;
  * `trigger_data` - N/A
* `callback_new` - This workflow is triggered when a new callback is created.
  * `trigger_data` - Dictionary with the following keys:
    * `payload_types` - a list of all the payload types where you _want_ this to trigger. If you don't specify any, then it will trigger for _all_ payload types.
* `task_intercept` - This workflow is triggered after a Task finishes its `opsec_post` check to allow one more chance for a task to be blocked.
  * `trigger_data` - N/A
* `response_intercept` - This workflow is triggered when a Task returns new output in the `user_output` field for the user to see in the UI, but first passes that output to this workflow for modification before saving it in the database.
  * `trigger_data` - N/A

