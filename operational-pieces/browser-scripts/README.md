# Browser Scripts

## What are they?

Browser Scripts allow users to script the output of agent commands. They are JavaScript functions that can return additional HTML for the webpage to parse for a cleaner experience.&#x20;

### Where are they?

Browser Scripts are located in the "Global Configurations" -> "Browser Scripts" section from the top navigation bar.

![User's Browser Scripts](<../../.gitbook/assets/Screen Shot 2020-07-13 at 10.06.00 AM.png>)

Every user has the default browser scripts automatically imported upon user creation.

### How are they applied?

Anybody can create their own browser scripts and they'll be applied only to that operator. If you create a script that's not tied to a specific command on a specific payload type, they are considered "Support Scripts" and are accessible to other scripts for that associated payload type. For example, in the above screenshot, the `create_table` script is accessible for other scripts in the apfell payload type to use to turn their output into a sortable table. These "Support Scripts" are callable from other scripts by prepending the payload type's name to the function name like: `support_scripts['apfell_create_table']`.&#x20;

You can also deactivate your own script so that you don't have to delete it, but it will no longer be applied to your output. This deactivates it globally and takes affect when the page is refreshed. For individual tasking you can use the tasking dropdown menu to toggle the affect on a single command.

If you are an overall admin or the lead of this operation, then you can optionally `Apply` the script to the operation. This forces your script to apply to the output of everybody's command in the operation and will override anybody else's script that would otherwise be applied. If you are curious as to whether your script will be in effect or not, the `In Effect` column indicates this.

The bottom table shows all of the scripts that are in effect for the current operation and will allow you to view the contents of the scripts even if you don't have permission to modify them.

### How are they created?

You can either import scripts that somebody else has created, or you can click `Register New Script` to create a new one. This is for one-off scripts you create. If you want to make it permanent across databases, then you need to add the script to the corresponding Payload Type's container. More information about that process can be found&#x20;

From the UI, you can tie the script to a specific command or select `Support Script` and provide a unique name to allow all scripts within the selected Payload Type to access it.. The editor here has a configurable theme, but has the language set to `JavaScript`. You can use this to create new scripts with syntax highlighting.

As you're creating a script, there are a few standard pieces:

* Don't supply a function name. All scripts are stored in a dictionary format of "name" to function, so simply start with `function`.
  * If you're creating a support script, you can create whatever parameters you want, but it should always be `function(param1, param2, ...)`.
  * If you're creating a command specific script, it will always be `function(task, responses)` where `task` is a JSON representation of the current task you're processing and `responses` is an array of `response` objects.
    * Response objects have two pieces of information:&#x20;
      * timestamp - the timestamp of when the response arrived
      * response - the contents of the actual response for the user to see. This will always be a string. If you actually returned JSON data back, be sure to run `JSON.parse` on this to convert it back to a JSON dictionary.
    * So, to access the first response's value, you'd say `responses[0]['response']`.
* Always return a value. From the example above you can see that you're able to return HTML that Mythic will display for you. It's recommended that you do proper error checking and handling.

### Toggling

Even if a browser script is pushed out for a command, its output can be toggled on and off individually.&#x20;
