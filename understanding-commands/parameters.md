# Parameters

Every command is different – some take no parameters, some take arrays, strings, integers, or a number of other things. To help accommodate this, you can add parameters to commands so that operators know what they need to be providing. You must give each parameter a name and they must be unique within that command. You can also indicate if the parameter is required or not.

You can then specify what type of parameter it is from the following:

* String
* CredentialJson
* Number
* Array
* Boolean
* Choose One
* Choose Many
* File
* PayloadList
* ConnectionInfo
* LinkInfo

If you select "PayloadList", the user is given a dropdown list of already created (but not deleted) payloads to choose from as a template. The associated tasking can then take this to fetch the payload contents, send down the payload's UUID, or use it to generate a new payload based on the one selected.

If you select "ConnectionInfo", the user is able to select and associate payloads/callbacks with hosts. This also allows the user to select the payload's p2p information for linking agents. The main example here is if you've generated a payload as part of lateral movement, priv esc, or persistence tasking and now want to connect to the payload via p2p (or if you lost connection and are reconnecting to the callback).

If you select "LinkInfo", the user gets a list of already established p2p "links" and allows you to select ones to re-establish or to remove.

Lastly, if you choose File, then the user will be allowed to upload a file via the GUI when you type out the command. By default, the file UUID is sent as the parameter so that you're ready to do file transfers via chunking.&#x20;

If a command takes named parameters, but none are supplied on the command line, a GUI modal will pop up to assist the operator.

There is no absolute requirement that the input parameters be in JSON format, it's just recommended.

{% hint style="info" %}
In order to modify the command or any of its components, you need to modify the corresponding python class in the Payload Type container.
{% endhint %}
