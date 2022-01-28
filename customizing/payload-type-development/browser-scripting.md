# Browser Scripting

## What is Browser Scripting?

Browser scripting is a way for you, the agent developer or the operator, to dynamically adjust the output that an agent reports back. You can turn data into tables, download buttons, screenshot viewers, and even buttons for additional tasking.

## Where are Browser Scripts?

As a developer, your browser scripts live in a folder called `browser_scripts` in your `mythic` folder. These are simply JavaScript files that you then reference from within your command files such as:

```python
browser_script = BrowserScript(script_name="download_new", author="@its_a_feature_", for_new_ui=True)
```

As an operator, they exist in the web interface under "Operations" -> "Browser Scripts". You can enable and disable these for yourself at any time, and you can even modify or create new ones from the web interface as well. If you want these changes to be persistent or available for a new Mythic instance, you need to save it off to disk and reference it via the above method.

{% hint style="info" %}
You need to supply `for_new_ui=True` in order for the script to be leveraged for the new user interface. If you don't do this, Mythic will attach the script to the old user interface. All of the following documentation is for the new user interface.
{% endhint %}

## What can I do via Browser Scripts?

Browser Scripts are JavaScript files that take in a reference to the task and an array of the responses available, then returns a Dictionary representing what you'd like Mythic to render on your behalf. This is pretty easy if your agent returns structured output that you can then parse and process. If you return unstructured output, you can still manipulate it, but it will just be harder for you.

### Plaintext

The most basic thing you can do is return plaintext for Mythic to display for you. Let's take an example that simply aggregates all of the response data and asks Mythic to display it:

```javascript
function(task, responses){
    const combined = responses.reduce( (prev, cur) => {
            return prev + cur;
        }, "");
    return {'plaintext': combined};
}
```

This function reduces the Array called `responses` and aggregates all of the responses into one string called `combined` then asks Mythic to render it via: `{'plaintext': combined}`.

Plaintext is also used when you don't have a browserscript set for a command in general or when you toggle it off. This uses the react-ace text editor to present the data. This view will also try to parse the output as JSON and, if it can, will re-render the output in pretty print format.

![](<../../.gitbook/assets/Screen Shot 2021-12-02 at 11.16.40 AM.png>)

### Screenshots

A slightly more complex example is to render a button for Mythic to display a screenshot.

```javascript
function(task, responses){
    if(task.status.toLowercase().includes("error")){
        const combined = responses.reduce( (prev, cur) => {
            return prev + cur;
        }, "");
        return {'plaintext': combined};
    }else if(task.completed){
        if(responses.length > 0){
            let data = JSON.parse(responses[0]);
            return {"screenshot":[{
                "agent_file_id": [data["agent_file_id"]],
                "variant": "contained",
                "name": "View Screenshot",
                "hoverText": "View screenshot in modal"
            }]};
        }else{
            return {"plaintext": "No data to display..."}
        }

    }else if(task.status === "processed"){
        // this means we're still downloading
        if(responses.length > 0){
            let data = JSON.parse(responses[0]);
            return {"screenshot":[{
                    "agent_file_id": [data["agent_file_id"]],
                    "variant": "contained",
                    "name": "View Partial Screenshot",
                    "hoverText": "View partial screenshot in modal"
            }]};
        }
        return {"plaintext": "No data yet..."}
    }else{
        // this means we shouldn't have any output
        return {"plaintext": "Not response yet from agent..."}
    }
}
```

This function does a few things:

1. If the task status includes the word "error", then we don't want to process the response like our standard structured output because we returned some sort of error instead. In this case, we'll do the same thing we did in the first step and simply return all of the output as `plaintext`.
2. If the task is completed and isn't an error, then we can verify that we have our responses that we expect. In this case, we simply expect a single response with some of our data in it. The one piece of information that the browser script needs to render a screenshot is the `agent_file_id` or `file_id` of the screenshot you're trying to render. If you want to return this information from the agent, then this will be the same `file_id` that Mythic returns to you for transferring the file. If you display this information via `process_response` output from your agent, then you're likely to pull the file data via an RPC call, and in that case, you're looking for the `agent_file_id` value. You'll notice that this is an _array_ of identifiers. This allows you to supply multiple at once (for example: you took 5 screenshots over a few minutes or you took screenshots of multiple monitors) and Mythic will create a modal where you can easily click through all of them.
3. To actually create a screenshot, we return a dictionary with a key called `screenshot` that has an array of Dictionaries. We do this so that you can actually render multiple screenshots at once (such as if you fetched information for multiple monitors at a time). For each screenshot, you just need three pieces of information: the `agent_file_id`, the `name` of the button you want to render, and the `variant` is how you want the button presented (`contained` is a solid button and `outlined` is just an outline for the button).
4. If we didn't error and we're not done, then the status will be `processed`. In that case, if we have data we want to also display the partial screenshot, but if we have no responses yet, then we want to just inform the user that we don't have anything yet.

### Downloads

When downloading files from a target computer, the agent will go through a series of steps to register a file id with Mythic and then start chunking and transferring data. At the end of this though, it's super nice if the user is able to click a button in-line with the tasking to download the resulting file(s) instead of then having to go to another page to download it. This is where the download browser script functionality comes into play.&#x20;

With this script, you're able to specify some plaintext along with a button that links to the file you just downloaded. However, remember that browser scripts run in the browser and are based on the data that's sent to the user to view. So, if the agent doesn't send back the new `agent_file_id` for the file, then you won't be able to link to it in the UI. Let's take an example and look at what this means:

```javascript
function(task, responses){
    if(task.status.includes("error")){
        const combined = responses.reduce( (prev, cur) => {
            return prev + cur;
        }, "");
        return {'plaintext': combined};
    }else if(task.completed){
        if(responses.length > 0){
            try{
                let data = JSON.parse(responses[0]);
                return {"download":[{
                        "agent_file_id": data["file_id"],
                        "variant": "contained",
                        "name": "Download",
                        "plaintext": "Download the file here: ",
                        "hoverText": "download the file"
                }]};
            }catch(error){
                const combined = responses.reduce( (prev, cur) => {
                    return prev + cur;
                }, "");
                return {'plaintext': combined};
            }

        }else{
            return {"plaintext": "No data to display..."}
        }

    }else if(task.status === "processed"){
        if(responses.length > 0){
            const task_data = JSON.parse(responses[0]);
            return {"plaintext": "Downloading a file with " + task_data["total_chunks"] + " total chunks..."};
        }
        return {"plaintext": "No data yet..."}
    }else{
        // this means we shouldn't have any output
        return {"plaintext": "Not response yet from agent..."}
    }
}
```

So, from above we can see a few things going on:

* Like many other browser scripts, we're going to modify what we display to the user based on the status of the task as well as if the agent has returned anything for us to view or not. That's why there's checks based on the `task.status` and `task.completed` fields.
* Assuming the agent returned something back and we completed successfully, we're going to parse what the agent sent back as JSON and look for the `file_id` field.
* We can then make the download button with a few fields:&#x20;
  * `agent_file_id` is the file UUID of the file we're going to download through the UI
  * `variant` allows you to control if the button is a solid or just outlined button (`contained` or `outlined`)
  * `name` is the text inside the button
  * `plaintext` is any leading text data you want to dispaly to the user instead of just a single download button

So, let's look at what the agent actually sent for this message as well as what this looks like visually:

![](<../../.gitbook/assets/Screen Shot 2021-12-16 at 7.25.16 PM.png>)

Notice here in what the agent sends back that there are two main important pieces: `file_id` which we use to pass in as `agent_file_id` for the browser script, and `total_chunks`. `total_chunks` isn't strictly necessary for anything, but if you look back at the script, you'll see that we display that as plaintext to the user while we're waiting for the download to finish so that the user has some sort of idea how long it'll take (is it 1 chunk, 5, 50, etc).

![](<../../.gitbook/assets/Screen Shot 2021-12-16 at 7.24.46 PM.png>)

And here you can see that we have our plaintext leading up to our button. You'll also notice how the `download` key is an array. So yes, if you're downloading multiple files, as long as you can keep track of the responses you're getting back from your agent, you can render and show multiple download buttons.

### Search Links

Sometimes you'll want to link back to the "search" page (tasks, files, screenshots, tokens, credentials, etc) with specific pieces of information so that the user can see a list of information more cleanly. For example, maybe you run a command that generated a lot of credentials (like mimikatz) and rather than registering them all with Mythic _and_ displaying them in the task output, you'd rather register them with Mythic and then link the user over to them. Thats where the search links come into play. They're formatted very similar to the download button, but with a slight tweak.

```javascript
function(task, responses){
    if(task.status.includes("error")){
        const combined = responses.reduce( (prev, cur) => {
            return prev + cur;
        }, "");
        return {'plaintext': combined};
    }else if(task.completed){
        if(responses.length > 0){
            try{
                let data = JSON.parse(responses[0]);
                return {"search": [{
                        "plaintext": "View on the search page here: ",
                        "hoverText": "opens a new search page",
                        "search": "tab=files&searchField=Filename&search=" + task.display_params,
                        "name": "Click Me!"
                    }]};
            }catch(error){
                const combined = responses.reduce( (prev, cur) => {
                    return prev + cur;
                }, "");
                return {'plaintext': combined};
            }

        }else{
            return {"plaintext": "No data to display..."}
        }

    }else if(task.status === "processed"){
        if(responses.length > 0){
            const task_data = JSON.parse(responses[0]);
            return {"plaintext": "Downloading a file with " + task_data["total_chunks"] + " total chunks..."};
        }
        return {"plaintext": "No data yet..."}
    }else{
        // this means we shouldn't have any output
        return {"plaintext": "Not response yet from agent..."}
    }
}
```

This is almost exactly the same as the `download` example, but the actual dictionary we're returning is a little different. Specifically, we have:

* `plaintext` as a string we want to display before our actual link to the search page
* `hoverText` as a string for what to display as a tooltip when you hover over the link to the search page
* `search` is the actual query parameters for the search we want to do. In this case, we're showing that we want to be on the `files` tab, with the `searchField` of `Filename`, and we want the actual `search` parameter to be what is shown to the user in the display parameters (`display_params`). If you're ever curious about what you should include here for your specific search, whenever you're clicking around on the search page, the URL will update to reflect what's being shown. So, you can navigate to what you'd want, then copy and paste it here.
* `name` is the text represented that is the link to the search page.

Just like with the download button, you can have multiple of these `search` responses.

![](<../../.gitbook/assets/Screen Shot 2021-12-16 at 7.35.38 PM.png>)

### Tables

Creating tables is a little more complicated, but not by much. The biggest thing to consider is that you're asking Mythic to create a table for you, so there's a few pieces of information that Mythic needs such as what are the headers, are there any custom styles you want to apply to the rows or specific cells, what are the rows and what's the column value per row, in each cell do you want to display data, a button, issue more tasking, etc. So, while it might seem overwhelming at first, it's really nothing too crazy.

Let's take an example and then work through it - we're going to render the following screenshot

![File Listing Browser Script](<../../.gitbook/assets/Screen Shot 2021-10-10 at 5.10.33 PM.png>)

```javascript
function(task, responses){
    if(task.status.includes("error")){
        const combined = responses.reduce( (prev, cur) => {
            return prev + cur;
        }, "");
        return {'plaintext': combined};
    }else if(task.completed && responses.length > 0){
        let folder = {
                    backgroundColor: "mediumpurple",
                    color: "white"
                };
        let file = {};
        let data = "";
        try{
            data = JSON.parse(responses[0]);
        }catch(error){
           const combined = responses.reduce( (prev, cur) => {
                return prev + cur;
            }, "");
            return {'plaintext': combined};
        }
        let ls_path = "";
        if(data["parent_path"] === "/"){
            ls_path = data["parent_path"] + data["name"];
        }else{
            ls_path = data["parent_path"] + "/" + data["name"];
        }
        let headers = [
            {"plaintext": "name", "type": "string", "fillWidth": true},
            {"plaintext": "size", "type": "size", "width": 200,},
            {"plaintext": "owner", "type": "string", "width": 400},
            {"plaintext": "group", "type": "string", "width": 400},
            {"plaintext": "posix", "type": "string", "width": 100},
            {"plaintext": "xattr", "type": "button", "width": 100, "disableSort": true},
            {"plaintext": "DL", "type": "button", "width": 100, "disableSort": true},
            {"plaintext": "LS", "type": "button", "width": 100, "disableSort": true},
            {"plaintext": "CS", "type": "button", "width": 100, "disableSort": true}
        ];
        let rows = [{
            "rowStyle": data["is_file"] ? file : folder,
            "name": {
                "plaintext": data["name"],
                "copyIcon": true,
                "startIcon": data["is_file"] ? "file":"openFolder",
                "startIconHoverText": data["name",
                "startIconColor": "gold",
                },
            "size": {"plaintext": data["size"]},
            "owner": {"plaintext": data["permissions"]["owner"]},
            "group": {"plaintext": data["permissions"]["group"]},
            "posix": {"plaintext": data["permissions"]["posix"]},
            "xattr": {"button": {
                "name": "View XATTRs",
                "type": "dictionary",
                "value": data["permissions"],
                "leftColumnTitle": "XATTR",
                "rightColumnTitle": "Values",
                "title": "Viewing XATTRs",
                "hoverText": "View more attributes"
            }},
            "DL": {"button": {
              "name": "DL",
              "type": "task",
              "disabled": !data["is_file"],
              "ui_feature": "file_browser:download",
              "parameters": ls_path
            }},
            "LS": {"button": {
                "name": "LS",
                "type": "task",
                "ui_feature": "file_browser:list",
                "parameters": ls_path
            }},
            "CS": {"button": {
                "name": "CS",
                "type": "task",
                "ui_feature": "code_signatures:list",
                "parameters": ls_path
                }}
        }];
        for(let i = 0; i < data["files"].length; i++){
            let ls_path = "";
            if(data["parent_path"] === "/"){
                ls_path = data["parent_path"] + data["name"] + "/" + data["files"][i]["name"];
            }else{
                ls_path = data["parent_path"] + "/" + data["name"] + "/" + data["files"][i]["name"];
            }
            let row = {
                "rowStyle": data["files"][i]["is_file"] ? file:  folder,
                "name": {"plaintext": data["files"][i]["name"]},
                "size": {"plaintext": data["files"][i]["size"]},
                "owner": {"plaintext": data["files"][i]["permissions"]["owner"]},
                "group": {"plaintext": data["files"][i]["permissions"]["group"]},
                "posix": {"plaintext": data["files"][i]["permissions"]["posix"],
                    "cellStyle": {

                    }
                },
                "xattr": {"button": {
                    "name": "View XATTRs",
                    "type": "dictionary",
                    "value": data["files"][i]["permissions"],
                    "leftColumnTitle": "XATTR",
                    "rightColumnTitle": "Values",
                    "title": "Viewing XATTRs"
                }},
                "DL": {"button": {
                  "name": "DL",
                  "type": "task",
                    "disabled": !data["files"][i]["is_file"],
                  "ui_feature": "file_browser:download",
                  "parameters": ls_path
                }},
                "LS": {"button": {
                    "name": "LS",
                    "type": "task",
                    "ui_feature": "file_browser:list",
                    "parameters": ls_path
                }},
                "CS": {"button": {
                    "name": "CS",
                    "type": "task",
                    "ui_feature": "code_signatures:list",
                    "parameters": ls_path
                }}
            };
            rows.push(row);
        }
        return {"table":[{
            "headers": headers,
            "rows": rows,
            "title": "File Listing Data"
        }]};
    }else if(task.status === "processed"){
        // this means we're still downloading
        return {"plaintext": "Only have partial data so far..."}
    }else{
        // this means we shouldn't have any output
        return {"plaintext": "Not response yet from agent..."}
    }
}
```

This looks like a lot, but it's nothing crazy - there's just a bunch of error handling and dealing with parsing errors or task errors. Let's break this down into a few easier to digest pieces:

```javascript
return {"table":[{
            "headers": headers,
            "rows": rows,
            "title": "File Listing Data"
        }]};
```

In the end, we're returning a dictionary with the key `table` which has an array of Dictionaries. This means that you can have multiple tables if you want. For each one, we need three things: information about headers, the rows, and the title of the table itself. Not too bad right? Let's dive into the headers:

```javascript
let headers =[
            {"plaintext": "name", "type": "string", "fillWidth": true},
            {"plaintext": "size", "type": "size", "width": 200,},
            {"plaintext": "owner", "type": "string", "width": 400},
            {"plaintext": "group", "type": "string", "width": 400},
            {"plaintext": "posix", "type": "string", "width": 100},
            {"plaintext": "xattr", "type": "button", "width": 100, "disableSort": true},
            {"plaintext": "DL", "type": "button", "width": 100, "disableSort": true},
            {"plaintext": "LS", "type": "button", "width": 100, "disableSort": true},
            {"plaintext": "CS", "type": "button", "width": 100, "disableSort": true}
        ];
```

Headers is an array of Dictionaries with three values each - `plaintext`, `type`, and optionally `width`. As you might expect, `plaintext` is the value that we'll actually use for the title of the column. `type` is controlling what kind of data will be displayed in that column's cells. There are a few options here: `string` (just displays a standard string), `size` (takes a size in bytes and converts it into something human readable - i.e. 1024 -> 1KB), `date` (process date values and display them and sort them properly), `number` (display numbers and sort them properly), and finally `button` (display a button of some form that does something). The last value here is `width` - this is a pixel value of how much width you want the column to take up by default. If you want one or more columns to take up the remaining widths, specify `"fillWidth": true`. Columns by default allow for sorting, but this doesn't always make sense. If you want to disable this ( for example, for a button column), set `"disableSort": true` in the header information.

Now let's look at the actual rows to display:

```javascript
        let rows = [{
            "rowStyle": data["is_file"] ? file : folder,
            "name": {
                "plaintext": data["name"],
                "copyIcon": true,
                "startIcon": data["is_file"] ? "file":"openFolder",
                "startIconHoverText": data["name",
                "startIconColor": "gold",
            },
            "size": {"plaintext": data["size"]},
            "owner": {"plaintext": data["permissions"]["owner"]},
            "group": {"plaintext": data["permissions"]["group"]},
            "posix": {"plaintext": data["permissions"]["posix"]},
            "xattr": {"button": {
                "name": "View XATTRs",
                "type": "dictionary",
                "value": data["permissions"],
                "leftColumnTitle": "XATTR",
                "rightColumnTitle": "Values",
                "title": "Viewing XATTRs",
                "hoverText": "View more attributes"
            }},
            "DL": {"button": {
              "name": "DL",
              "type": "task",
              "disabled": !data["is_file"],
              "ui_feature": "file_browser:download",
              "parameters": ls_path
            }},
            "LS": {"button": {
                "name": "LS",
                "type": "task",
                "ui_feature": "file_browser:list",
                "parameters": ls_path
            }},
            "CS": {"button": {
                "name": "CS",
                "type": "task",
                "ui_feature": "code_signatures:list",
                "parameters": ls_path
                }}
        }];
```

Ok, lots of things going on here, so let's break it down:

#### rowStyle

As you might expect, you can use this key to specify custom styles for the row overall. In this example, we're adjusting the display based on if the current row is for a file or a folder.

#### plaintext

If we're displaying anything other than a button for a column, then we need to include the `plaintext` key with the value we're going to use. You'll notice that aside from `rowStyle`, each of these other keys match up with the `plaintext` header values so that we know which values go in which columns.

In addition to just specifying the `plaintext` value that is going to be displayed, there are a few other properties we can specify:

* `startIcon` - specify the name of an icon to use at the beginning of the `plaintext` value. The available `startIcon` values are:
  * folder/openFolder, closedFolder, archive/zip, diskimage, executable, word, excel, powerpoint, pdf/adobe, database, key, code/source, download, upload, png/jpg/image, kill, inject, camera, list, delete
  * ^ the above values also apply to the `endIcon` attribute
* `startIconHoverText` - this is text you want to appear when the user hovers over the icon
* `endIcon` this is the same as the `startIcon` except it's at the end of the text
* `endIconHoverText` this is the text you want to appear when the user hovers over the icon
* `plaintextHoverText` this is the text you want to appear when the user hovers over the plaintext value
* `copyIcon` - use this to indicate true/false if you want a `copy` icon to appear at the front of the text. If this is present, this will allow the user to copy all of the text in `plaintext` to the clipboard. This is handy if you're displaying exceptionally long pieces of information.
* `startIconColor` - You can specify the color for your start icon. You can either do a color name, like `"gold"` or you can do an rgb value like `"rgb(25,142,117)"`.&#x20;
* `endIconColor` - this is the same as the `startIconColor` but applies to any icon you want to have at the end of your text

#### dictionary button

The first kind of button we can do is just a popup to display additional information that doesn't fit within the table. In this example, we're displaying all of Apple's extended attributes via an additional popup.

```javascript
{
    "button": 
    {
        "name": "View XATTRs",
        "type": "dictionary",
        "value": data["permissions"],
        "leftColumnTitle": "XATTR",
        "rightColumnTitle": "Values",
        "title": "Viewing XATTRs",
        "hoverText": "View additional attributes"
    }
}
```

The button field takes a few values, but nothing crazy. `name` is the name of the button you want to display to the user. the `type` field is what kind of button we're going to display - in this case we use `dictionary` to indicate that we're going to display a dictionary of information to the user. The other type is `task` that we'll cover next. The `value` here should be a Dictionary value that we want to display. We'll display the dictionary as a table where the first column is the key and the second column is the value, so we can provide the column titles we want to use. We can optionally make this button disabled by providing a `disabled` field with a value of `true`. Just like with the normal `plaintext` section, we can also specify `startIcon`, `startIconColor.`Lastly, we provide a `title` field for what we want to title the overall popup for the user.

#### string button

If the data you want to display to the user isn't structured (not a dictionary, not an array), then you probably just want to display it as a string. This is pretty common if you have long file paths or other data you want to display but don't fit nicely in a table form.&#x20;

```json
{
    "button": 
    {
        "name": "View Strings",
        "type": "string",
        "value": "my data string\nwith newlines as well",
        "title": "Viewing XATTRs",
        "hoverText": "View additional attributes"
    }
}
```

Just like with the other button types, we can use `startIcon`, `startIconColor`, and `hoverText` for this button as well.

#### task button

This button type allows you to issue additional tasking.

```javascript
{
    "button": 
    {
              "name": "DL",
              "type": "task",
              "disabled": !data["is_file"],
              "ui_feature": "file_browser:download",
              "parameters": ls_path,
              "hoverText": "List information about the file/folder"
      }
  }
```

This button has the same `name` and `type` fields as the dictionary button. Just like with the dictionary button we can make the button disabled or not with the `disabled` field. You might be wondering which task we'll invoke with the button. This works the same way we identify which command to issue via the file browser or the process browser - `ui_feature`. These can be anything you want, just make sure you have the corresponding feature listed somewhere in your commands or you'll never be able to task it. Just like with the dictionary button, we can specify `startIcon` and `startIconColor`.&#x20;

The last thing here is the `parameters`. If you provide parameters, then Mythic will automatically use them when tasking. In this example, we're pre-creating the full path for the files in question and passing that along as the parameters to the `download` function. If you don't provide any parameters and the task you're trying to issue takes parameters, then you will get a popup to provide the parameters, just like if you tasked it from the command line.

#### table button

Sometimes the data you want to display is an array rather than a dictionary or big string blob. In this case, you can use the `table` button type and provide all of the same data you did when creating this table to create a new table (yes, you can even have menu buttons on that table).

```json
{
    "button":
    {
        "name": "view table",
        "type": "table",
        "title": "my custom new table",
        "value": {
            "headers": [
                {"plaintext": "test1", "width": 100, "type": "string"}, {"plaintext": "Test2", "type": "string"}
            ],
            "rows": [
                {"test1": {"plaintext": "row1 col 1"}, "Test2": {"plaintext": "row 1 col 2"}}
            ]
        }
    }
}
```

#### menu button

Tasking and extra data display button is nice and all, but if you have a lot of options, you don't want to have to waste all that valuable text space with buttons. To help with that, there's one more type of button we can do: `menu`. With this we can wrap the other kinds of buttons:

```json
"button": {
    "name": "Actions",
    "type": "menu",
    "value": [
            {
                "name": "View XATTRs",
                "type": "dictionary",
                "value": data["files"][i]["permissions"],
                "leftColumnTitle": "XATTR",
                "rightColumnTitle": "Values",
                "title": "Viewing XATTRs"
            },
            {
                "name": "Get Code Signatures",
                "type": "task",
                "ui_feature": "code_signatures:list",
                "parameters": ls_path
            },
            {
                "name": "LS Path",
                "type": "task",
                "ui_feature": "file_browser:list",
                "parameters": ls_path
            },
            {
              "name": "Download File",
              "type": "task",
              "disabled": !data["files"][i]["is_file"],
              "ui_feature": "file_browser:download",
              "parameters": ls_path
            }
        ]
    }
```

Notice how we have the exact same information for the `task` and `dictionary` buttons as before, but they're just in an array format now. It's as easy as that. You can even keep your logic for disabling entries or conditionally not even add them. This allows us to create a dropdown menu like the following screenshot:

![](<../../.gitbook/assets/Screen Shot 2021-11-04 at 3.03.58 PM.png>)

These menu items also support the `startIcon` , `startIconColor` , and `hoverText`, properties.
