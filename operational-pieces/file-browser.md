---
description: Unified, Persistent File Browser
---

# File Browser

## What is it?

The file browser is a visual, file browser representation of the directory listings that agents perform. Not all agents support this feature however.

## Where is it?

From any callback dropdown in the "Active Callbacks" window, select "File Browser" and the view will be rendered in the lower-half of the screen. This information is a combination of the data across _all_ of the callbacks, and is persistent.

![File Browser View](<../.gitbook/assets/Screen Shot 2020-08-09 at 7.22.28 PM.png>)

## How do you use it?

The view is divided into two pieces - a graphical hierarchy on the left and a more detailed view of a folder on the right. The top layer on the left will be the hostname and everything below it will correspond to the file structure for that host.&#x20;

You'll notice a green checkmark for the `files` folder. The green checkmark means that an agent reported back information for that folder specifically (i.e. somebody tasked an `ls` of that folder or issued a `list` command via the button on the table side). This is in contrast the other folders in that tree - those folders are "implicitly" known because we have the full path returned for the folder we did access. If there is a red circle with an exclamation point, it means that you tried to perform an `ls` on the directory, but it failed.

On the right hand side, the table view has a few pieces along the top:

* The first field is the `hostname` associated with the file path. This will default to the host associated with the path, but can be changed separately to allow list of hosts not currently in the view.
* The second field is the `path` associated with the information below. If you haven't received any information from any agent yet or you haven't clicked on a path, this will default to the current directory `.`.&#x20;
* The third item is the `list` button. This looks at the far right hand side Callback number, finds the associated payload type, then looks for the command with `is_file_browse` set to `true`. Then issues that command with the `host` and `path` shown in the first two fields. If you want to list the contents of a directory that you can't see in the UI, just modify these two values and hit `list`.
* Similar to the 3rd item, the 4th item is the `upload` button. This will look for the `is_upload` field for the payload type associated with the identified Callback and execute that command. In most cases this will cause a popup dialog where you can upload your file.
* The last field is the `Callback` field. As you browse this data, it's aggregated across all of the callbacks and hosts that are reporting data back. Therefore, you might have some information from Callback 1, some from Callback 15, etc. When you click on a file or folder, this value will change to the Callback where the data came from.
  * If you want to list a known path from a _different_ callback than the one that returned the data you're looking at, simply change this number to the number of the new callback and click `list` or `upload`.

## Actions

For each entry in the table menu on the right, there are some actions you can do:

![File Browser Actions](<../.gitbook/assets/Screen Shot 2020-08-09 at 7.33.17 PM.png>)

The file browser only shows _some_ information that's returned. There are portions that are Operating Specific though - like UNIX permissions, extended attributes, or SDDLs. This information doesn't make sense to display in the main table, so clicking the `View Permissions` action will display a popup with more specific information.

The `Download History` button will display information about all the times that file has been downloaded. This is useful when you repeatedly download the same file over and over again (ex: downloading a user's Chrome Cookie's file every day). If you've downloaded a file, there will be a green download icon next to the filename. This will always point to the _latest_ version of the file, but you can use the `download history` option to view all other instances in an easy pane. This popup will also show the comments associated with the tasks that issued the download commands.

The other three are self explanatory - tasking to list a file/folder, download a file, or remove a file/folder. If a file is removed and reports back the removal to hook into the file browser, then the filename will have a small trash icon next to it and the name will have a strikethrough.
