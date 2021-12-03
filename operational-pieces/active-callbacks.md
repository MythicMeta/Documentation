# Active Callbacks

## Where is it?

The main page to see and interactive with active callbacks can be found from the phone icon at the top of the screen.

### Top table

The top table has a list of current callbacks with a bunch of identifying information. All of the table headers can be clicked to sort the information in ascending or descending order.

![](<../.gitbook/assets/Screen Shot 2021-12-02 at 5.05.27 PM.png>)

* Callback - The identifying callback number. The blue or red button will bring the bottom section into focus, load the previously issued tasks for that callback, and populate the bottom section with the appropriate information (discussed in the next section).
  * If the `integrity_level` of the callback is <= 2, then the callback button will be blue. Otherwise it'll be red (indicating high integrity) and there will be an `*` next to the username. It's up to the agent to report back its own integrity level
* Host - The hostname for the machine the callback is from
* IP - The IP associated with the host
* User - The current user context of the callback
* PID - The process ID for the callback
* OS (arch) - This is the OS and architecture information for the host
* Initial Checkin - The time when the callback first checked in. This date is stored in UTC in the database, but converted to the operator's local time zone on the page.
* Last Checkin - How long it's been since the last checkin in day:hour:minute:second time\\
* Description - The current description of the callback. The default value for this is specified by the `default tag` section when creating a payload. This can be changed either via the callback's dropdown or after clicking the keyboard button, type `set description whatever you want here`.

Next to the `Interact` button is a dropdown button that provides more accessible information:

![](<../.gitbook/assets/Screen Shot 2021-12-02 at 4.47.01 PM.png>)

* Expand Callback - This opens up the callback in a separate window where you can either just view that whole callback full screen, or selectively add other callbacks to view in a split view
* Edit Description - This allows you to edit the description of a callback. This will change the side description at the end and also rename the tab at the bottom when somebody clicks `interact`. To set this back to the default value, interact with the callback and type `set description reset`. or set this to an empty string
* Hide Callback - This removes the callback from the current view and sets it to inactive. Additionally, from the Search page, you can make the callback `Active` again which will bring it back into view here.
* Process Browser - This allows you to view a unified process listing from all agents related to this `host`, but issue new process listing requests from within this callback's context
* Locked - If a callback is locked by a specific user, this will be indicated here (along with a changed user and lock icon instead of a keyboard on the interacting button).
* File Browser - this allows you to view a process browser across all of the agents.

## Bottom Area

The bottom area is where you'll find the tasks, process listings, file browsers, and comments related to specific callbacks. Clicking the keyboard icon on a callback will open or select the corresponding tab in this area.

### Auto Complete

When you start typing a command, you can press `Tab` to finish out and cycle through the matching commands. If you don't type anything and hit `Tab` then you'll cycle through all available commands. You can use the up and down arrow keys to cycle through the tasking history for that callback, and you can use `ctrl+r` to do a reverse grep search through your previous history as well.

### Tasking

Submitting a command goes through a few phases that are also color coded to help visually see the state of your task:

1. Preprocessing - This is when the command is submitted to Mythic, but execution is passed to the associated Payload Type's command file for processing. These capabilities are covered in more depth in the Payload Types section.
2. Submitted- The task has finished pre-processing and is ready for the agent to request it.
3. Processing - The agent has pulled down the task, but has not returned anything.
4. Processed - The agent has returned _at least one_ response for the task, but hasn't explicitly marked the task as completed
5. Completed - The agent has reported the task done successfully
6. Error -The agent reported that there was an error with executing the task.

Once you've submitted tasking, there's a bit of information that'll be automatically displayed.

* The user that submitted the task
* The task number - You can click on this task number to view just that task and its output in a separate page. This makes it easy to share the output of a task between members of an operation.
* The command and any parameters supplied by the operator

#### Task filtering

The very bottom right hand of the screen has a little filter button that you can click to filter out what you see in your callbacks. The filtering only applies as long as you're on that callback page (i.e. it gets reset when you refresh the page).

![](<../.gitbook/assets/Screen Shot 2021-12-02 at 4.57.18 PM.png>)
