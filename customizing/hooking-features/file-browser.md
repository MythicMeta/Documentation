# File Browser

## Components

For the file browser, there are a few capabilities that need to be present and implemented correctly if you want to allow users to list, remove, download, or upload from the file browser. Specifically:

* File Listing - there needs to be a command marked as `supported_ui_features = ["file_browser:list"]` with your payload type that sends information in the proper format.
* File Removal - there needs to be a command marked as `supported_ui_features = ["file_browser:remove"]` with your payload type that reports back success properly
* File Download - there needs to be a command marked as `supported_ui_features = ["file_browser:download"]` with your payload type
* File Upload - there needs to be a command marked as `supported_ui_features = ["file_browser:upload"]` with your payload type

These components together allow an operator to browse the file system, request listings of new directories, track downloaded files, upload new files, and even remove files. Let's go into each of the components and see what they need to do specifically.

### File Listing

There are two components to file listing that need to be handled - what the file browser sends as initial tasking to the command marked as `supported_ui_features = ["file_browser:list"]`and what data is sent back to Mythic for processing.

#### Tasking

When doing a file listing via the file browser, the command\_line for tasking will always be the following as as JSON string (this is what gets sent as the `self.command_line` argument in your command's `parse_arguments` function):

```python
{
  "host": "hostname of computer to list",
  "path": "path to the parent folder",
  "file": "name of the file or folder you're trying to list",
  "full_path": "absolute path to the file/folder"
}
```

This might be different than the normal parameters you take for the command marked as `supported_ui_features = ["file_browser:list"]`. Since the payload type's command handles the processing of arguments itself, we can handle this case and transform the parameters as needed. For example, the `apfell` payload takes a single parameter `path` as an argument for file listing, but that doesn't match up with what the file browser sends. So, we can modify it within the `async def parse_arguments` function:

```python
async def parse_arguments(self):
    if len(self.command_line) > 0:
        if self.command_line[0] == '{':
            temp_json = json.loads(self.command_line)
            if 'host' in temp_json:
                # this means we have tasking from the file browser rather than the popup UI
                # the apfell agent doesn't currently have the ability to do _remote_ listings, so we ignore it
                self.add_arg("path", temp_json['path'] + "/" + temp_json['file'])
            else:
                self.add_arg("path", temp_json['path'])
            self.add_arg("file_browser", "true")
        else:
            self.add_arg("path", self.command_line)
            self.add_arg("file_browser", "true")
```

In the above example we check if we are given a JSON string or not by checking that the `self.command_line` length is greater than 0 and that the first character is a `{`. We can then parse it into a Python dictionary and check for the two cases. If we're given something with `host` in it, then it must come from the file browser instead of the operator normally, so we take the supplied parameters and add them to what the command normally needs. In this case, since we only have the one argument `path`, we take the `path` and `file` variables from the file browser dictionary and combine them for our path variable.

#### Agent File Browsing Responses

Now that we know how to translate file browsing file listing tasking to whatever our command needs, what kind of output do we need to send back?

We have another component to the `post_response` for agents.

```python
{
    "action": "post_response",
    "responses": [
        {
            "task_id": "UUID of task",
            "user_output": "file browser issued listing", // optional
            "file_browser": {
                "host": "hostname of computer you're listing", // optional
                "is_file": True or False,
                "permissions": {json of permission values you want to present},
                "name": "name of the file or folder you're listing",
                "parent_path": "full path of the parent folder",
                "success": True or False,
                "access_time": unix epoc time in milliseconds,
                "modify_time": unix epoc time in milliseconds,
                "size": 1345, //size of the entity
                "update_deleted": True, //optional
                "files": [ // if this is a folder, include data on the files within
                    {
                        "is_file": True or False,
                        "permissions": {json of data here that you want to show},
                        "name": "name of the entity",
                        "access_time": unix epoc time in milliseconds,
                        "modify_time": unix epoc time in milliseconds,
                        "size": 13567 // size of the entity
                    }
                ]
            }
        }
    ]
}
```

{% hint style="info" %}
As a shortcut, if the file you're removing is on the same host as your callback, then you can omit the `host` field or set it to `""` and Mythic will automatically add in your callback's host information instead.
{% endhint %}

{% hint style="info" %}
If you're listing out the top level folder (`/` on linux/macOS or a drive like `C:\` on Windows, then the parent path should be "" or null.
{% endhint %}

Most of this is pretty self-explanatory, but there are some nuances. Only list out the inner files for the initial folder/file listed (i.e. don't recursively do this listing). For the `files` array, you don't need to include `host` or `parent_path` because those are both inferred based on the info outside the `files` array, and the `success` flag won't be included since you haven't tried to actually list out the contents of any sub-folders. The permissions JSON blob allows you to include any additional information you want to show the user in the file browser. For example, with the `apfell` agent, this blob includes information about extended attributes, posix file permissions, and user/group information. Because this is heavily OS specific, there's no requirement here other than it being a JSON blob (not a string).

By having this information in _another_ component within the responses array, you can display any information to the user that you want without being forced to _also_ display this listing each time to the user. You can if you want, but it's not required. If you wanted to do that, you could simply turn all of the `file_browser` data into a JSON string and put it in the `user_output` field. In the above example, the user output is a simple message stating why the tasking was issued, but it could be anything (even left blank).

{% hint style="info" %}
Mythic doesn't currently support `.`, `..`, or `~` paths. Any information about `.` should be part of the main `file_browser` JSON data (not part of the `files` array). `~` should be fixed to an absolute path.
{% endhint %}

#### update deleted

There's a special key in there that doesn't really match the rest of the normal "file" data in that file\_browser response - `update_deleted`. If you include this key as `True` and your `success` flag is `True`, then Mythic will use the data presented here to update which files are deleted.

By default, if you list the contents of `~/Downloads` twice, then the view you see in the UI is a _merge_ of all the data from those two instance of listing that folder. However, that might not _always_ be what you want. For instance, if a file was deleted between the first and second listing, that deletion won't be reflected in the UI because the data is simply merged together. If you want that delete to be automatically picked up and reported as a deleted file, use the `update_deleted` flag to say to Mythic "hey, this should be everything that's in the folder, if you have something else that used to be there but I'm not reporting back right now, assume it's deleted".

You might be wondering why this isn't just the default behavior for listing files. There are two main other scenarios that we want to support that are counter to this idea - paginated results (only return 20 files at a time) and filtered results (only return files in the folder that end in .txt). In these cases, we don't want the rest of the data to be automatically marked as deleted because we're clearly not returning the full picture of what's in a folder. That's why it's an optional flag to say to performing the automatic updates. If you want to be explicit with things though (for example, if you delete a file and want to report it back without having to re-list the entire contents of the directory), you can use the next section - FIle Removal.

### File Removal

There are two components to file listing that need to be handled - what the file browser sends as initial tasking to the command marked as `supported_ui_features = ["file_browser:remove"]`and what data is sent back to Mythic for processing.

#### Tasking

This is the exact same as the `supported_ui_features = ["file_browser:list"]`and the [File Browser](file-browser.md#tasking) section above.

#### Agent File Removal Responses

Ok, so we listed files and tasked one for removal. Now, how is that removed file tracked back to the file browsing to mark it as removed? Nothing too crazy, there's another field in the `post_response`:

```python
{
    "action": "post_response",
    "responses": [
        {
            "task_id": "UUID of task",
            "user_output": "File successfully deleted",
            "removed_files": [
                {
                    "host": "hostname where file was removed",
                    "path": "full path to the file"
                }
            ]
        }
    ]
}
```

This `removed_files` section simply returns an array of dictionaries that spell out the host and paths of the files that were deleted. On the back-end, Mythic takes these two pieces of information and searches the file browsing data to see if there's a matching `path` for the specified `host` in the current operation that it knows about. If there is, it gets marked as `deleted` and in the UI you'll see a small trashcan next to it along with a strikethrough.

This response isn't ONLY for when a file is removed through the file browser though. You can return this from your normal removal commands as well and if there happens to be a matching file in the browser, it'll get marked as removed. This allows you to simply type things like `rm /path/to/file` on the command-line and still have this information tracked in the file browser without requiring you to remove it through the file browser specifically.

### File Downloading

There are two components to file listing that need to be handled - what the file browser sends as initial tasking to the command marked as `supported_ui_features = ["file_browser:download"]`and what data is sent back to Mythic for processing.

#### Tasking

This is the exact same as the `supported_ui_features = ["file_browser:list"]` and the [File Browser](file-browser.md#tasking) section above.

#### Agent File Download Responses

There's nothing special here outside of normal file download processes described in the [Download](download.md) section. When a new file is tracked within Mythic, there's a "host" field in addition to the `full_path` that is reported. This information is used to look up if there's any matching browser objects and if so, they're linked together.

### File Uploading

Having a command marked as `supported_ui_features = ["file_browser:upload"]`will cause that command's parameters to pop-up and allow the operator to supply in the normal file upload information. To see this information reflected in the file browser, the user will then need to issue a new file listing.
