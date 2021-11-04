# Active Callbacks

## Where is it?

The main page to see and interactive with active callbacks can be found from the "Operational Views" -> "Active Callbacks" page from the top navigation bar.

### Top table

The top table has a list of current callbacks with a bunch of identifying information. All of the table headers can be clicked to sort the information in ascending or descending order.

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
* Filtering - This top bar allows you to filter what information is presented in the callback pane. This is specified by `column`:`search value`. For example, to only display callbacks from a certain host, use `host:spooky` .
  * The available columns are: id, host, ip, user, domain, os, payload\_type, integrity\_level

Next to the `Interact` button is a dropdown button that provides more accessible information:

![](<../.gitbook/assets/Screen Shot 2020-07-13 at 10.41.43 AM.png>)

* Keystrokes - This opens up an additional tab at the bottom for keystrokes related to that callback
* Screenshots - This opens up an additional tab at the bottom for screenshots related to that callback
* Expand Callback - This opens up the callback in a separate window where you can either just view that whole callback full screen, or selectively add other callbacks to view in a split view
* Edit Description - This allows you to edit the description of a callback. This will change the side description at the end and also rename the tab at the bottom when somebody clicks `interact`. To set this back to the default value, interact with the callback and type `set description reset`. or set this to an empty string
* Exit Callback - Sends the `exit` command to the callback. Because this can be different for each agent type, Mythic looks up the command that has `is_exit` set to `True` for the right payload type, and sends that command.
* Remove Callback - This removes the callback from the current view and sets it to inactive. This can still be viewed in the `All Callbacks` page (covered in a later section). If a callback is removed and checks back in, it'll automatically be brought back into view on this screen. Additionally, from the `All Callbacks` page, you can make the callback `Active` again which will bring it back into view here.
* Processes - This allows you to view a unified process listing from all agents related to this `host`, but issue new process listing requests from within this callback's context
* Locked - If a callback is locked by a specific user, this will be indicated here (along with a changed user and lock icon instead of a keyboard on the interacting button).&#x20;

## Bottom Area

The bottom area is where you'll find the tasks, keylogs, screenshots, process listings, and comments related to specific callbacks. Clicking the keyboard icon on a callback will open or select the corresponding tab in this area.

The current agent you're interact with is highlighted at the top and bottom. The tasking bar at the very bottom will be auto populated with the `user@hostname(pid)` of the callback you're tasking.

### Auto Complete

When you start typing a command, the possible commands that match what you're typing will be shown above the tasking line. You can use the up and down arrow keys to toggle through these values. Hitting `Enter` or `tab` will finish the command on the command line.

### Tasking&#x20;

Submitting a command goes through a few phases that are also color coded to help visually see the state of your task:

1. Preprocessing - This is when the command is submitted to Mythic, but execution is passed to the associated Payload Type's command file for processing. These capabilities are covered in more depth in the Payload Types section.
2. Submitted- The task has finished pre-processing and is ready for the agent to request it.
3. Processing - The agent has pulled down the task, but has not returned anything.
4. Processed - The agent has returned _at least one_ response for the task, but hasn't explicitly marked the task as completed
5. Completed - The agent has reported the task done successfully
6. Error -The agent reported that there was an error with executing the task.

Once you've submitted tasking, there's a bit of information that'll be automatically displayed.&#x20;

* The user that submitted the task
* The task number - You can click on this task number to view just that task and its output in a separate page. This makes it easy to share the output of a task between members of an operation.
* The command and any parameters supplied by the operator
* The status dropdown can be selected to show more options such as downloading that task's output, copying the command, viewing any parameters modified by the tasking function, disabling any associated browser scripts, and even adding comments to the task

As you scroll through your tasks, the current task will be fixed to the top. This is helpful when there's a lot of task output and you don't want to scroll all the way to the top of the task to minimize it.

#### Task filtering

The very bottom right hand of the screen has a little filter button that you can click to filter out what you see in your callbacks. The filtering only applies as long as you're on that callback page (i.e. it gets reset when you refresh the page), but allows you to filter by:

* Operator - user and operator's name (or part of one) to only display tasking from names that contain what you type
* Task Numbers - select a lower and upper bound (inclusive on both ends) for the range of tasks you want to see
* Command - only show commands of a certain type (like only shell commands, only ls, etc)

You can do as many or as few of these together as you want, and the icon in the bottom right will indicate that some are active by changing to a yellow color.

![Filtering tasks](<../.gitbook/assets/Screen Shot 2020-02-17 at 9.33.39 PM.png>)
