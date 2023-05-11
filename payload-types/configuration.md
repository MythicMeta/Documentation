# Configuration

## View Configuration

All of a payload type's parameters are configurable from within the Payload Type's docker container. Edit the corresponding information in the class that extends the PayloadType class.

```python
class Apfell(PayloadType):

    name = "apfell"
    file_extension = "js"
    author = "@its_a_feature_"
    supported_os = [
        SupportedOS.MacOS
    ]
    wrapper = False
    wrapped_payloads = []
    note = """This payload uses JavaScript for Automation (JXA) for execution on macOS boxes. 
There have been issues with macOS < 10.13 and encryption, so be sure to blank out the AESPSK and set key exchange to "F" when creating those payloads.
    """
    supports_dynamic_loading = True
    build_parameters = {
    }
    c2_profiles = ["http", "dynamichttp"]

```

There are a few interesting pieces to call out here:

* "Is this payload a wrapper for another payload"
  * A "Payload Wrapper" is a special form of payload type that simply acts a a wrapper for another payload type. An easy example of this is `msbuild` or `macros` from the Windows environment. These are payloads you might drop onto a system, but they aren't the _real_ payload you're trying to execute. They're just wrappers for the actual end payload. That's the same goal here.
* Does this payload support dynamic loading - This is where you can specify if your payload allows you to load new modules in it or not. If this is false, then when creating a payload, you will not be able to choose which commands you want stamped into it - they'll _ALL_ always be stamped in. If this is set to true, it does allow dynamic loading, then you can freely choose which commands you want stamped in at creation time and load in new commands later.

<figure><img src="../.gitbook/assets/Screenshot 2023-03-06 at 12.26.11 PM.png" alt=""><figcaption></figcaption></figure>
