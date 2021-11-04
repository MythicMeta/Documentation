# Scripting Files

## What does this hook into?

Scripting tasking involves the following RESTful endpoints on an instance of `Mythic`. This means you need to create a new `Mythic` instance (i.e. `mythic = Mythic(username="blah" ...` ) and then call these functions like `mythic.download_file()`:

```python
    async def download_file(self, file: FileMeta) -> bytes:
        """
        Download a file that is either scheduled for upload or is finished downloading
        """
```

The `FileMeta` class type refers to this: [https://github.com/MythicMeta/Mythic\_Scripting/blob/master/mythic/mythic\_rest.py#L2801](https://github.com/MythicMeta/Mythic\_Scripting/blob/master/mythic/mythic\_rest.py#L2801). All of the scripting tries to work on Objects rather than opaque dictionaries, so if you want to download a file, you need to indicate which file you want to download. Let's take an example:

```python
await mythic.listen_for_new_files(analyze_file_upload_download)
...
async def analyze_file_upload_download(mythic, file):
    try:
        if file.total_chunks == file.chunks_received:
            if file.is_download_from_agent:
                print("[+] Notified of finished file download, pulling from server for offline analysis...")
                contents = await mythic.download_file(file)
                with open("downloaded_file", "wb") as f:
                    f.write(contents)
            else:
                print("this is an upload or screenshot")

        else:
            print(f"[*] Don't have full file yet: {file.chunks_received} of {file.total_chunks} so far")
    except Exception as e:
        print(e)
```
