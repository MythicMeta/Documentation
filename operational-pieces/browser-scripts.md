# Browser Scripts

## What are they?

Browser Scripts allow users to script the output of agent commands. They are JavaScript functions that can return structured data to indicate for the React user interface to generate tables, buttons, and more.

### Where are they?

Browser Scripts are located in the hamburger icon in the top left -> "Operations" -> BrowserScripts.

Every user has the default browser scripts automatically imported upon user creation based on which agents are installed.

![](<../.gitbook/assets/Screen Shot 2021-12-02 at 3.46.05 PM.png>)

### How are they applied?

Anybody can create their own browser scripts and they'll be applied only to that operator. You can also deactivate your own script so that you don't have to delete it, but it will no longer be applied to your output. This deactivates it globally and takes affect when the task is toggled open/close. For individual tasking you can use the speed dial at the bottom of the task and select to "Toggle Browserscript".

If you are an overall admin or the lead of this operation, then you can optionally `Apply` the script to the operation. This forces your script to apply to the output of everybody's command in the operation and will override anybody else's script that would otherwise be applied. If you are curious as to whether your script will be in effect or not, the `In Effect` column indicates this.

The bottom table shows all of the scripts that are in effect for the current operation and will allow you to view the contents of the scripts even if you don't have permission to modify them.

### How are they created?

Click `Register New Script` to create a new one. This is for one-off scripts you create. If you want to make it permanent across databases and for other operators, then you need to add the script to the corresponding Payload Type's container. More information about that process can be found here: [#what-is-browser-scripting](../customizing/payload-type-development/browser-scripting.md#what-is-browser-scripting "mention").

When you're creating a script, the function declaration will always be `function(task, responses)` where `task` is a JSON representation of the current task you're processing and `responses` is an array of the responses displayed to the user. This will always be a string. If you actually returned JSON data back, be sure to run `JSON.parse` on this to convert it back to a JSON dictionary. So, to access the first response value, you'd say `responses[0]`.

You should always return a value. It's recommended that you do proper error checking and handling. You can check the status of the task by looking at the `task` variable and checking the `status` and `completed` attributes.

### Toggling

Even if a browser script is pushed out for a command, its output can be toggled on and off individually.
