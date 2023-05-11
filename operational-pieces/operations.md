# Operations

## What is an operation?

Operations are collections of operators, payloads, tasks, artifacts, callbacks, and files. While payload types and c2 profiles are shared across an entire Mythic instance, operations allow fine grained control over the visibility and access during an assessment.

## Where are operations?

Operation information can be found via the hamburger icon in the top left, then selecting "Operations" -> "Modify Operations" page. If you're a global Mythic admin, you'll see _all_ operations here. Otherwise, you'll only see operations that are associated with your account. Only a global Mythic admin can create new operations.

![](<../.gitbook/assets/Screen Shot 2021-12-02 at 3.27.10 PM.png>)

## How do you use operations?

Every operation has at least one member - the lead operator. Other operators can be assigned to the operation with varied levels of access.

* `operator` is your normal user.
* `lead` is the lead of that operation
* `spectator` can't do anything within Mythic. They essentially have Read-Only access across the entire operation. They can't create payloads, issue tasking, add comments, send messages, etc. They can search and view callbacks/tasking, but that's it.

For more fine-grained control than that listed above, you can also create _block lists_. These are named lists of commands that an operator is _not allowed to execute_ for a specific payload type. These block lists are then tied to specific operators. This offers a middle-ground between normal operator with full access and a spectator with no access. You can edit these block lists via the yellow edit button.

For the configure button for the operation, there are many options. You can specify a Slack webhook along with the channel. By default, whenever you create a payload via the "Create Payloads" page, it is tagged as alert-able - any time a new callback is created based on that payload, this slack webhook will be invoked. If you want to prevent that for a specific payload, go to the payloads page, select the "Actions" dropdown for the payload in question, and select to stop alerting. If you have the Slack webhook set on the operation overall, other payloads will continue to generate alerts, but not the ones you manually disable. You can always enable this feature again in the same way.

For the operators edit button, you can edit who is assigned to the operation, what their roles are, and specify which (if any) block lists should be assigned to that user.

![](<../.gitbook/assets/Screen Shot 2021-12-02 at 3.31.27 PM.png>)

## Current Operations

Because many aspects of an assessment are tied to a specific operation (payloads, callbacks, tasks, files, artifacts, etc), there are many things that will appear empty within the Mythic UI until you have an operation selected as your current operation. This lets the Mythic back-end know which data to fetch for you. If you don't have an operation as your active one, then you'll see no operation name listed on the top center of your screen. Go to the operations page and, if you're assigned to an operation that you can see, you can select to "Make Current". This process will require you to log out and log back in for the effect to take place and the new data to be fetched.
