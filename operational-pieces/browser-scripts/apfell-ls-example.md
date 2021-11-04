# apfell ls example

This section will walkthrough an example of how to write a browser script for a command and leverage support scripts.

## Command Output

The first step is to understand the command output that you will be scripting. This is a lot easier to do if you can make the output structured in some fashion. Apfell agents tend to do JSON output, but this isn't a hard requirement.

For the `ls` command with the `apfell` agent, the output looks like:

```
{
      "NSFileExtensionHidden": false,
      "NSFileCreationDate": "2017-06-26T18:04:55.000Z",
      "NSFileReferenceCount": 6,
      "NSFileModificationDate": "2019-07-27T21:45:07.269Z",
      "type": "D",
      "files": [
            {
                  "name": "test.txt",
                  "type": "",
                  "size": 1475,
                  "permissions": "644",
                  "owner": "itsafeature(501)",
                  "group": "staff(20)",
                  "hidden": ""
            }
      ],
      "name": "/Users/itsafeature/Desktop/",
      "size": 192,
      "permissions": "700",
      "owner": "itsafeature(501)",
      "group": "staff(20)",
      "hidden": ""
}
```

So, there is information about the folder or file that `ls` was run against, and if it's a folder, there's an array of extra information in a `files` keyword.&#x20;

## Creating a Script

When creating a script, it's important to understand the flow of what gets passed to your script and when it gets called. Scripts are loaded in to the browser when you request certain pages and are applied to tasks every time. So, if you have a callback with three `ls` commands, the `ls` browser script will be applied to all of them. Refresh the page and it'll be applied to all of them again.&#x20;

The next step is to start creating the function:

```javascript
function(task, responses){  // this will be the same for all scripts tied to a specific command
  var output = "";
  for(var i = 0; i < responses.length; i++){
    try{
        var data = JSON.parse(responses[i]['response']);
        output += JSON.stringify(data, null, 2);
    }catch(error){
        return "Failed to parse json with error: " + error.toString() + "\n" + JSON.stringify(response, null, 2);
    }
  }
  return output;
}
```

Let's start examining the above basic script:

* `function(task, response)`&#x20;
  * Since this is tied to a specific command, it'll always have this same definition. The first parameter will be a JSON dictionary of the task and the second parameter will be an array of JSON dictionaries of the responses.
* The next thing is just looping through the responses. For a basic starting point and knowing that the output will be JSON, try to just parse the actual response value into JSON and convert it back to a string to display. If everything is parsed correctly, then our output from this should look the same as if there was no browser script applied at all.

### Displaying Output

The next step is to decide what to do with the output. The easiest and most common scenario will be to display the output in a table, but you're able to do whatever. Since this is likely to be a common task, it's good to create a `support script` to create tables in general, then simply leverage that script with our command's specific data.

There is already a `support_script` for the apfell agent called `create_table` that takes in two arguments:

1. An array of dictionaries that specify the `name` of a column and the size of the column in `rem` element sizes. By default ,the table spans the width of the screen, so sometimes the columns will be bigger than specified
2. An array of dictionaries that specify the content and styling of the rows.

Judging from our sample output, an appropriate call to this support script might look like the following:

```javascript
support_scripts['apfell_create_table'](
[
{"name": "name", "size": "10em"},
{"name": "size", "size": "2em"}, 
{"name": "permissions", "size": "2em"},
{"name": "owner","size": "3em"},
{"name": "group", "size": "2em"},
{"name": "hidden", "size": "1em"},
{"name": "type", "size": "1em"}
], rows);
```

Now to actually get the data for the rows.

```javascript
var row_style = "";
// to the columns "hidden" and "type", style them to be "text-align:center"
var cell_style = {"hidden": "text-align:center",
                  "type":"text-align:center"};
// push information about the initial file
rows.push({"name": data['name'],
           "size": data['size'],
           "permissions": data['permissions'],
           "owner": data['owner'],
           "group": data['group'],
           "hidden": data['hidden'],
           "type": data['type'
           "row-style": row_style,
           "cell-style": cell_style
});
// check if ls of directory or file
if(!data.hasOwnProperty('files')){data['files'] = []};
// iterate through all the files and add them to the rows for the table
data['files'].forEach(function(r){
  var row_style = "";
// if the filename has .sh in it, make the row red with white text
  if(r['name'].includes(".sh")){row_style="background-color:red;color:white"}
  rows.push({"name": r['name'],
             "size": r['size'],
             "permissions": r['permissions'],
             "owner": r['owner'],
             "group": r['group'],
             "hidden": r['hidden'],
             "type": r['type'],
             "row-style": row_style,
             "cell-style": {"hidden": "text-align:center",
                            "type":"text-align:center"}
            });
});
```

Now that all of the data is parsed out and formatted for the `create_table` support script, it can be applied and tested. Save your script and you'll see a toast notification that it was successfully updated. Now, refresh a page where you'd see the output of the script and you should see your code come into play. If there are errors, they will be JavaScript errors, so it's helpful to have the debug console open so you can see your browser's console window and check for errors.&#x20;

### Final Script and Output

```javascript
function(task, responses){
	if(task.status === 'error'){
		return "<pre> Error: untoggle for error message(s) </pre>";
	}
  let rows = [];
  try{
  	for(let i = 0; i < responses.length; i++){
        let data = JSON.parse(responses[i]['response']);
        let row_style = "";
		let cell_style = {"hidden": "text-align:center", "type":"text-align:center"};
	    rows.push({"name": data['name'],
                   "size": data['size'],
                   "permissions": data['permissions'],
                   "owner": data['owner'],
                   "group": data['group'],
                   "type": data['type'],
                   "xattr": "",
                   "row-style": row_style,
                   "cell-style": cell_style
        });
	    if(!data.hasOwnProperty('files')){data['files'] = []};
	    data['files'].forEach(function(r){
    		let row_style = "";
    		//if(r['name'].includes(".sh")){row_style="background-color:red;color:white"}
    		if(r.hasOwnProperty('ExtendedAttributes')){
    			r['xattr'] = Object.keys(r['ExtendedAttributes']).join("<br>");
    		}else{
    			r['xattr'] = "";
    		}
		    rows.push({"name": r['name'],
	                   "size": r['size'],
	                   "permissions": r['permissions'],
	                   "owner": r['owner'],
	                   "group": r['group'],
	                   "type": r['type'],
	                   "xattr": r['xattr'],
	                   "row-style": row_style,
	                   "cell-style": {"hidden": "text-align:center", "type":"text-align:center"}
           });
		});
	}
    	return support_scripts['apfell_create_table']([{"name":"name", "size":"10em"},{"name":"size", "size":"2em"}, {"name":"permissions", "size":"2em"},{"name":"owner","size":"3em"} ,{"name": "group", "size": "2em"} ,{"name":"type", "size": "1em"}, {"name":"xattr", "size": "1em"}], rows);
    }catch(error){
        return  "<pre> Error: untoggle for error message(s) </pre>";
    }
}
```

