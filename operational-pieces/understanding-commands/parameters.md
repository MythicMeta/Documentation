# Parameters

Every command is different – some take no parameters, some take arrays, strings, integers, or a number of other things. To help accommodate this, you can add parameters to commands so that operators know what they need to be providing. You must give each parameter a name and they must be unique within that command. You can also indicate if the parameter is required or not.

Parameters can be in conditional parameter "groups" - this allows you to say things like parameter X and parameter Y are mutually exclusive, but you should always supply parameter W. As an operator, if there are any parameter groups for a command and you don't provide enough parameters to determine which group to use, Mythic will throw a warning and ask you to use `shift + enter` to force the modal popup. From here, there's a dropdown at the top to change the group you're looking at to see which parameters to enter.

If a command takes named parameters, but none are supplied on the command line, a GUI modal will pop up to assist the operator.

There is no absolute requirement that the input parameters be in JSON format, it's just recommended.

{% hint style="info" %}
In order to modify the command or any of its components, you need to modify the corresponding python class in the Payload Type container.
{% endhint %}
