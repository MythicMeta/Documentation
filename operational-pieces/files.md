# Files

## Where is it?

All uploads and downloads for an operation can be tracked via the "Operational Views" -> "Files" page from the top navigational bar.

## How does it work?

This page simply shows all uploads and downloads tracked by Mythic and breaks them up by task.

![](<../.gitbook/assets/Screen Shot 2020-02-17 at 10.00.31 PM.png>)

From here, you can see who download or uploaded a file, when it happened, and where it went to or came from. Clicking the download button will download the file to the user's machine.

If you want to download multiple files at once from the `Downloads` section, click the toggle for all the files you want and select the `Zip & Download selected` button at the top right to download them.

Files are saved as random UUID values on disk with a mapping to the real names in the database. This doesn't come into effect when you download a single file, but when you zip and download files the filenames will stay the UUIDs. To help with this, inside that zip is a `mappings.json` file with a mapping of real names to UUIDs. This is done as a security precaution to prevent malicious file names/paths.&#x20;

### Additional Info

Each file has additional information such as the SHA1 and MD5 of each file that can be viewed by clicking the blue info icon. If there's a comment on the task associated with the file upload or download, that comment will be visible here as well.

### Exporting

You can export the metadata about all of the files via the export function for Uploads and Downloads respectively. This includes all of the information you see in the UI, but just doesn't include the actual files themselves.
