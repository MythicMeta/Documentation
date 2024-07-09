# Operator Context (run\_as)

## Event workflow actions as ...?

If you're thinking about creating an eventing workflow, but curious who these actions would run as, then you're in the right spot! It shouldn't be the case where something takes actions as you without you knowing it. Mythic tracks everything that's happening, so if an action happens on your behalf that you didn't authorize, then it starts to break down that model. Instead, there's the `run_as` field in the workflow file and required consent. Let's break it down.

## bot

Every time an operation is created, a "bot" account is also created and added to the operation as a standard "operator". More detail on the accounts can be found [here](../../../operators.md#bots). The default operational context for a workflow is the "bot" account for the operation. However, we don't want just anybody to upload workflows to do arbitrary things within Mythic. That could get dangerous. Instead, anybody can upload the workflow files, but if the execution context is for the bot, then the admin of the operation must approve it to run.

<figure><img src="../../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

At any point, the admin of the operation can go back and change prior approval to a deny or a deny to an approval. The last time it changed will always be tracked. Also, if the admin of the operation changes, then all of their prior approvals are removed and the new admin must re-approve them.

Since the "bot" account is a normal operator, you can apply block lists as well. That makes it easy to block what scripts are able to execute vs what you want to require a human intervention to task. Bots are always easy to identify when assigning operators because they have a robot symbol next to their name:&#x20;

<figure><img src="../../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
If you leave `run_as` blank or omit it entirely, then `bot` is the default value used.
{% endhint %}

## self

A `run_as` value of `self` means that the workflow will execute under the context of the operator that uploaded it.

## trigger

A `run_as` value of `trigger` means that the workflow will execute under the context of the operator that triggered it (or bot if there wasn't an explicit trigger). For this case, each operator must provide their consent or it'll fail to run for operators that don't provide consent.

## lead

A `run_as` value of `lead` means that the workflow will execute under the context of the operation admin. Naturally, the operation admin must approve this before this can execute.

## anything else

If you supply a value to `run_as` that's none of the above header values (bot, self, trigger, lead), then it's assumed that you're trying to run within the context of a specific operator. If the name matches an existing operator, then that operator must be part of the operation and have granted consent. If you specify the name of a bot, then the lead of the operation must grant consent first.
