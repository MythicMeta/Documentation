---
description: Unified, Persistent File Browser
---

# File Browser

## What is it?

The file browser is a visual, file browser representation of the directory listings that agents perform. Not all agents support this feature however.

## Where is it?

From any callback dropdown in the "Active Callbacks" window, select "File Browser" and the view will be rendered in the lower-half of the screen. This information is a combination of the data across _all_ of the callbacks, and is persistent.

![File Browser View](<../.gitbook/assets/Screenshot 2023-03-06 at 8.42.40 AM.png>)

## How do you use it?

The view is divided into two pieces - a graphical hierarchy on the left and a more detailed view of a folder on the right. The top layer on the left will be the hostname and everything below it will correspond to the file structure for that host.

You'll notice a green checkmark for the `files` folder. The green checkmark means that an agent reported back information for that folder specifically (i.e. somebody tasked an `ls` of that folder or issued a `list` command via the button on the table side). This is in contrast the other folders in that tree - those folders are "implicitly" known because we have the full path returned for the folder we did access. If there is a red circle with an exclamation point, it means that you tried to perform an `ls` on the directory, but it failed.

On the right hand side, the table view has a few pieces along the top:

* The text field is the `path` associated with the information below with the corresponding hostname right above it. If you haven't received any information from any agent yet or you haven't clicked on a path, this will default to the current directory `.`.
* The first button is the `list` button. This looks at the far right hand side Callback number, finds the associated payload type, then looks for the command with `is_file_browse` set to `true`. Then issues that command with the `host` and `path` shown in the first two fields. If you want to list the contents of a directory that you can't see in the UI, just modify these two values and hit `list`.
* The second button is the `upload` button. This will look for the `is_upload` field for the payload type associated with the identified Callback and execute that command. In most cases this will cause a popup dialog where you can upload your file.
* The last field allows you to toggle viewing deleted files or not.

## Actions

For each entry in the table menu on the right, there are some actions you can do by clicking the gear icon:

![File Browser Actions](<../.gitbook/assets/Screenshot 2023-03-06 at 8.45.32 AM.png>)

The file browser only shows _some_ information that's returned. There are portions that are Operating Specific though - like UNIX permissions, extended attributes, or SDDLs. This information doesn't make sense to display in the main table, so clicking the `View Permissions` action will display a popup with more specific information.

The `Download History` button will display information about all the times that file has been downloaded. This is useful when you repeatedly download the same file over and over again (ex: downloading a user's Chrome Cookie's file every day). If you've downloaded a file, there will be a green download icon next to the filename. This will always point to the _latest_ version of the file, but you can use the `download history` option to view all other instances in an easy pane. This popup will also show the comments associated with the tasks that issued the download commands.

The other three are self explanatory - tasking to list a file/folder, download a file, or remove a file/folder. If a file is removed and reports back the removal to hook into the file browser, then the filename will have a small trash icon next to it and the name will have a strikethrough.
