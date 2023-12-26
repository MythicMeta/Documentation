# Change Log

What's new for 2.3?

* [x] New React-based User Interface
* [x] New Parameter group based command parsing when tasking
  * [x] Users can tab-complete command parameters
  * [x] Parameter Groups allow for conditional parameter sets
  * [x] Tasking tracks if issued from command line, parsed cli, modals, browser scripts
* [x] Updated MITRE ATT\&CK mappings
* [x] Reporting updated to use LaTeX for PDF generation
  * [x] can generate PDF, raw LaTeX, or JSON output
  * [x] Reporting can now filter out callbacks, users, or hosts
  * [x] Reporting has an option for a MITRE ATT\&CK overview with counts at the end
* [x] Browser Scripting updated
  * [x] Browser scripts now return dictionaries instead of raw HTML
  * [x] Users can issue follow-on tasks right from browser scripts
  * [x] Browser Scripts can generate tables, download links, links to search pages, and screenshot buttons
* [x] Improved search page
  * [x] Can search across tasking, callbacks, files, screenshots, tokens, and more
  * [x] Searches can be saved and linked to via the updated URL bar
* [x] Translation containers now tracked in the UI and indicated when they are associated or not associated to payload type containers
* [x] Improved information for C2 profile status (container running vs internal server running)
* [x] Payloads page shows which C2 profiles are loaded into the payloads and if those profiles are currently running or not
* [x] You can re-trigger a payload to build from the payloads page without having to go through all the steps again
* [x] C2 profiles can expose redirect rules and check agent configurations to make sure everything aligns well between the two. These functions exist within the C2 Profile container and are taskable through the payloads page in the UI.
* [x] The main callbacks page has new icons for the egress route of agents (wifi or link icon) that, when clicked, will show a graph view of the egress path of that agent's traffic.&#x20;
* [x] The main callbacks page can track the sleep information for a callback automatically (based on the agent updating it for the callback) and display it (orange or blue alarm clock icon)
* [x] All columns on the main callbacks page are filterable based on data or can be hidden completely by right clicking on the column headers.
  * [x] This also applies to all of the tables generated via browser scripting
* [x] Response output is now paginated to help reduce the stress of streaming a lot of output for agents
* [x] Response tables in from browser scripts are now virtualized to help reduce the stress of displaying hundreds or thousands of rows at a time
* [x] Normal output is now displayed via an Ace editor, so if the output is JSON, you can collapse/expand sections like a normal text editor
* [x] Graph view now uses Dagre for D3 rendering of graphs and automatically groups agents together on the same host for easier reading
  * [x] The display can be changed via the dropdown on the top left for:
    * [x] Left-to-right or bottom-to-top directions
    * [x] Showing all callbacks or just the active ones
    * [x] Showing all edges or only the active edges
    * [x] The resulting graph can be saved as a PNG for easier inclusion in reports
* [x] GraphQL endpoints have been added for the new React user interface
  * [ ] the scripting PyPi package will eventually move to use these instead of the REST endpoints
*

## What's new for 2.2?

* [x] "translation container" support for agent's that want their own custom c2 message spec and support for arbitrary other encryption outside of the default Mythic supported ones
* [x] OPSEC scripting checks, logging, bypasses for tasking
* [x] Solidified P2P spec - [Delegates](../customizing/c2-related-development/c2-profile-code/agent-side-coding/delegates.md) and [P2P Connections](../customizing/hooking-features/linking-agents/action-p2p\_info.md)
* [x] Alpha version of new back-end/UI updates for nginx reverse proxy, hasura graphql, and react
* [x] All Payload Types moved out of main Mythic repo and into Mythic Agents organization
* [x] Track sleep information in UI in easy-to-see form (new UI only)
* [x] Track linked agent info in the table ui form of working with callbacks (new ui only)
* [x] New way of viewing linked agents in the UI using dagre-d3 instead of raw d3 (new ui only)
* [x] Commands track which OS they support. This helps things payload types like poseidon that might have os-specific commands since golang can compile to multiple operating systems
* [x] Deleted payloads can't be used to create new callbacks - instead, mythic will generate a warning in the UI alerting you with information about the payload, but the agent will get a 404 and blank message back.
* [x] De-duplication of alert messages
* [x] Tasks can change their `display_params` in the `create_tasking` function to alter what user's see - this helps with ugly large json blocks from using structured input. The original parameters can still be queried through the UI if the user wants to see the original JSON
* [x] Tasks can set stdout/stderr during `create_tasking` to help with debugging or recreating artifacts
* [x] Payload creation can track its own `build_message` , `build_stderr` , and `build_stdout`for stdout and stderr messages to make it easier to see what happened during creation. This split information is also viewable via the Created Payloads page.
* [x] Command parameters can opt to have their choices be all commands for a payload type or all loaded commands
* [x] Command parameters can filter down their choices based on command attributes (supported os and if they support process injection)
* [x] Password policy now added - requires all passwords to be at least 12 characters long
* [x] Mythic scripting (i.e. the `mythic` PyPi package) has been updated to version 0.0.19 to support these changes
* [x] C2 Profile specific OPSEC checks - provide an `opsec` function to your C2 Profiles to check if the parameters that the operator selected pass your risk-level before even allowing a payload to be created.
* [x] RPC Calls are updated within the Payload Type containers to allow for dynamic queries and updates. New RPC calls just import one file
* [x] Removed ability for users to create new accounts on the login page
* [x] Re-build payload option from the created payloads page
* [x] Export payload from created payloads page and import it in the payloads creation page to trigger a new payload
* [x] Starting Mythic with `MYTHIC_DEBUG` environment variable or `debug` set to true in the `mythic-docker/config.json` file will cause mythic to be extremely verbose in agent messages to help with debugging development
* [x] Merged in a pull request to update ATT\&CK to ATT\&CK v9

{% hint style="danger" %}
The following are the breaking changes for this version
{% endhint %}

* For detailed steps on how to make updates as a developer, check out the [updating guide](../updating/mythic-2.1-greater-than-2.2-updates/agents-2.1.-greater-than-2.2.2) for how to take your 2.1.\* agent into the 2.2.2 Mythic
* All Payload Types and C2 Profiles are split out from the main Mythic repo. If you just install mythic and start it, there will be _no_ payload types or c2 profiles that sync in. You need to leverage the installer script to install additional agents from GitHub
  * To help support this and keep track of these sorts of updates, containers now have "versions" and the main Mythic server has a range of supported container versions. If you have a container that tries to check in that's outside that range, you'll get a warning in the Mythic UI.
* All Payload Types need to do the following:
  * Control for container files have been moved into PyPi packages rather than just raw files on disk so that updating them will be easier going forward. The current version needed for this is version 0.0.42 (yes, lots of development has been going on): `pip install mythic_payloadtype_container==0.0.42`.
  * If you are using a DockerImage from the `itsafeaturemythic` repo, update to the latest:
    * csharp\_payload==0.0.11
    * python38\_payload==0.0.4
    * xgolang\_payload==0.0.9
    * leviathan\_payload==0.0.4
* Updating to using a PyPi package instead of local files means that you need to adjust the imports for all of the Python files for your agents. It's nothing too crazy, but you'll update like so: [Payload Type Info](../customizing/payload-type-development/payload-type-info.md#building)
* All C2 Profiles need to do the following:
  * Control for container files has been moved into PyPi packages rather than just raw files on disk so that updating them will be easier going forward. The current version needed for this is version 22: `pip install mythic_c2`\_container`==0.0.22`
* Updating to using a PyPi packages instead of local files means that you need to adjust the imports for all of the Python files for your C2 profiles. It's nothing too crazy, but you'll need to update like so if you've created your own C2 profiles: [C2 Docker Containers](../customizing/c2-related-development/c2-profile-code/server-side-coding/c2-docker-containers.md#c-2-profile-class).
* Mythic crypto is defined by the Payload Type now rather than the C2 profile. This is part of a change to keep each container in charge of just one thing. Payload Types by default have Mythic handle their crypto, to have a Payload Type do something _other_ than what Mythic does, set `mythic_encrypts = False` in the builder file for your Payload Type.
* To more easily support various crypto components going forward, the selection and usage of Crypto parameters in C2 profiles has changed slightly. There's no longer a requirement that the parameter name be `AESPSK`. Instead, there is another field you can specify for any parameter that `crypto_type=True`. This specifies that the resulting thing that the user selects/inputs defines what kind of crypto to use. This is simply a boolean value so that you can still leverage the C2 Parameter as normal (string input, choose one, etc) with the expectation that the final value is the type of crypto. In the case of Mythic's standard crypto, the value would be `aes256_hmac`. This means that Mythic no longer auto-generates a base64 of an AES256 key when displayed to the user, this happens behind the scenes if the resulting type is `aes256_hmac`.
  * This also causes some variation when building your payload. Normally, you get a dictionary of {"key": "value"} for each C2 Profile parameter for you to leverage when building your payload. For crypto though, this could be highly variable and the components that you want to leverage could vary widely. So, if the parameter has `crypto_type=True`, then you'll get a dictionary of values. This is split out by type, encryption key, and decryption key because you might want to leverage some pub/priv key asymmetric crypto where those pieces are different or you might want to leverage some other kind of symmetric crypto.

```python
{
    "key":
    {
        "crypto_type": "the value that the user selected, like aes256_hmac",
        "enc_key": "base64 of the encryption key",
        "dec_key": "base64 of the decryption key"
    }
}
```

* Make sure you update your pip install of the `mythic==0.0.19` package for scripting to handle the updated aspects of these objects.

## What Changed for 2.0?

* [x] Rebranded Apfell to Mythic
* [x] Added File Browser support to the apfell and poseidon agents
* [x] Added SOCKS5 support to poseidon
* [x] Added user roles during operations (operator, developer, spectator)
* [x] Added a documentation docker container with more verbose details including overviews, traffic flows, opsec considerations, and detailed help/info
* [x] Updated the general UI for Mythic
* [x] Updated the event logging system for "warnings" that can be resolved
* [x] Added ability for search to look through file browsing and eventing data as well
* [x] Operator feedback if their parameters don't meet certain validation requirements before the commands even make it to the agents
* [x] Complete restructuring of how a developer uses/creates agents/c2 profiles
* [x] Added RPC call functionality from payload type and c2 profile containers to the main mythic instance to start hooking into the back-end for scripting

## What changed for 1.4?

* [x] Toggle timestamp view in the UI to see either UTC timestamps or localized timestamps
* [x] Persistent, unified processes listings based on the host
* [ ] Persistent, unified file browser
* [x] Badges to see the number of new tasks on a callback you're not currently viewing
* [x] More malleable HTTP-based C2 profile with a JSON config
  * [x] Cookie Support
  * [x] Arbitrary transforms to text (base64, append, prepend, add random values, etc etc)
  * [ ] Proxy Support
  * [x] User Agent strings
* [x] Slack notifications on new callbacks
  * [x] The ability to have this only fire for specific payloads (i.e. you might not want it to happen for all of your lateral movement payloads, but do want it to fire for your phishing payloads)
* [x] Exportable search results (just export the page or the entire results of a search query)
* [x] Exportable artifact search results (just the page or the entire results)
* [x] More granular search controls
  * [x] Filter search by operator
* [x] Duplicate saved parameters for c2 profiles to make quick edits
* [x] Edit credentials that are saved in the database
* [x] Add comments to the saved credentials
* [x] Export final report in JSON format
* [x] Export information about downloaded / uploaded files
* [x] Automatically calculate MD5/SHA1 of files
* [x] Badge notifications and searching for Keylogging
* [x] Task comments shown when viewing files
* [x] Download command output as a text file
* [x] Selective caching of responses so large output doesn't slow down the entire callback
* [x] Confirmation for mass exit/remove via the UI of callbacks
* [x] Issuing a command that causes a parameter popup will first try to auto-populate the values with the last instance of the command you ran on that callback
* [x] Save current browsing view in main callback window across refreshes (open tabs)
* [x] 8hr default token expiration and browser will auto-renew periodically so that there are fewer interruptions in longer ops
* [x] Use tab to select autocomplete option and up/down arrows to toggle through autocomplete options
* [x] Use `ctrl+[` and `ctrl+]` to navigate previous/next tab in the `active callbacks` tab.
* [x] set callback description from command line via `set description my description` and reset it back to the default for the payload with `set description reset`
* [x] Your open tabs for the active callbacks view will be remembered per-browser
* [x] View Apfell's web log from within the main web UI
* [x] Only start select payload type containers when starting Apfell - `./start_apfell.sh viper` will only start the `viper` payload type container, but all of the c2 profiles and main containers will still start.
* [x] Turn arbitrary VMs into Apfell compatible containers
* [x] Import and export single commands at a time instead of the whole payload type
* [x] C2 profiles support jitter percentages
* [x] Kill Dates in C2 profiles
* [x] Bulk download files from downloads page as zip file
* [ ] Strict argument checking on tasking to make sure required parameters are given
* [x] Provide a single streaming list of tasking for all callbacks combined that the operation lead can watch
* [x] Filter callback viewable tasks to a single operator or all operators, certain command, and certain task ranges
* [x] Add new buttons for hiding or exiting multiple callbacks at once
* [x] C2 profiles have associated Notes, Sample Server configuration, and Sample Client configurations that are present from the main UI to give context and base configuration information. This will be particularly useful as C2 profiles grow more complex.
* [x] Provide and an interface for supporting P2P communications and visualizations
* [ ] Provide a recommended style guide for how to create new payloads to best fit in and leverage all of Apfell's features
* [x] Integration of Poseidon Agent
* [x] Integration of Atlas Agent
* [x] Integration of Chrome-extension Agent
* [x] Introduction of command-line short-hands such as swap\_filenames to allow operators to type uploaded filenames, but have agents get file IDs instead.
* [x] All files are now contained within the Apfell folder, no more docker volumes mounted in weird spots automatically
* [x] Edit C2 profile code and configurations right in the browser
* [x] Included an 'event log' to see all events Apfell is doing behind the scenes as well as allow operators to 'chat' and store messages within their operation. If an operator sends a message on this screen, it'll be shown to all operators in that operation that are on the Active Callbacks page.
* [x] Included a 'web log' to see all the web requests to Apfell in the UI
* [x] Broke out transforms for commands and create/load operations to mirror the style of browser scripts
* [x] Exporting/Importing a payload type will bring along with it: payload code, all associated transforms, all command code, all browser scripts, and all c2 profile associated code.
