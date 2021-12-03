# Files

## Where is it?

All uploads and downloads for an operation can be tracked via the clip icon or the search icon at the top.

## How does it work?

This page simply shows all uploads and downloads tracked by Mythic and breaks them up by task.

From here, you can see who download or uploaded a file, when it happened, and where it went to or came from. Clicking the download button will download the file to the user's machine.

If you want to download multiple files at once from the `Downloads` section, click the toggle for all the files you want and select the `Zip & Download selected` button at the top right to download them.

Files are saved as random UUID values on disk with a mapping to the real names in the database. This doesn't come into effect when you download a single file, but when you zip and download files the filenames will stay the UUIDs. To help with this, inside that zip is a `mappings.json` file with a mapping of real names to UUIDs. This is done as a security precaution to prevent malicious file names/paths.

![](<../.gitbook/assets/Screen Shot 2021-12-02 at 5.08.25 PM.png>)

### Additional Info

Each file has additional information such as the SHA1 and MD5 of each file that can be viewed by clicking the blue info icon. If there's a comment on the task associated with the file upload or download, that comment will be visible here as well.

