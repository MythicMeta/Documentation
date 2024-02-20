# 10. Translation Containers

### Translation Containers

If you want to have a different form of communication between Mythic and your agent than the specific JSON messages that Mythic uses, then you'll need a "translation container".&#x20;

The first thing you'll need to do is specify the name of the container in your associated Payload Type class code. Update the Payload Type's class to include a line like `translation_container = "binaryTranslator"` . Now we need to create the container.&#x20;

The process for making a translation container is almost identical to a c2 profile or payload type container, we're simply going to change which classes we instantiate, but the rest of it is the same.

Unlike Payload Type and C2 Profile containers that mainly do everything over RabbitMQ for potentially long-running queues of jobs, Translation containers use gRPC for fast responses.&#x20;

If a `translation_container` is specified for your Payload Type, then the three functions defined in the following two examples will be called as Mythic processes requests from your agent.

You then need to get the new container associated with the docker-compose file that Mythic uses, so run `sudo ./mythic-cli add binaryTranslator`. Now you can start the container with `sudo ./mythic-cli start binaryTranslator` and you should see the container pop up as a sub heading of your payload container.

Additionally, if you're leveraging a payload type that has `mythic_encrypts = False` and you're doing any cryptography, then you should use this same process and perform your encryption and decryption routines here. This is why Mythic provides you with the associated keys you generated for encryption, decryption, and which profile you're getting a message from.

{% hint style="info" %}
Mythic will base64 decode the message it gets, pull out the payload/staging/callback UUID in front, and look up information on it. When Mythic determines the backing payload type has a translation container, Mythic will send the UUID, encryption information, and encrypted blob to your translation container. When Mythic is done processing the message from your agent and it's time to send a response back, it'll send the message to your translation container and then forward the response back to the C2 Profile.&#x20;

**NOTE:** If your translation container is doing the encryption, then Mythic will expect that the message coming back from your translation container (mythic c2 to custom) will be encrypted, have the UUID attached, and be base64 encoded. Mythic will do **NOTHING** with the message it gets back. If Mythic is handling the encryption, then you will simply return the custom c2 bytes of your message and **MYTHIC** will be the one to add the UUID and base64 encode the response.
{% endhint %}

### Python

For the Python version, we simply instantiate our own subclass of the TranslationContainer class and provide three functions. In our `main.py` file, simply import the file with this definition and then start the service:&#x20;

```
mythic_container.mythic_service.start_and_run_forever()
```

{% @github-files/github-code-block url="https://github.com/MythicMeta/ExampleContainers/blob/main/Payload_Type/python_services/translator/translator.py" fullWidth="true" %}

### GoLang

For the GoLang side of things, we instantiate an instance of the translationstructs.TranslationContainer struct with our same three functions. For GoLang though, we have an Initialize function to add this struct as a new definition to track.&#x20;

{% @github-files/github-code-block url="https://github.com/MythicMeta/ExampleContainers/blob/main/Payload_Type/go_services/no_actual_translation/translationfunctions/translations.go" fullWidth="true" %}

Then, in our `main.go` code, we call the Initialize function and start the services:

```go
mytranslatorfunctions.Initialize()
// sync over definitions and listen
MythicContainer.StartAndRunForever([]MythicContainer.MythicServices{
   MythicContainer.MythicServiceTranslationContainer,
})
```

### Examples:

These examples can be found at the MythicMeta organization on GitHub: [https://github.com/MythicMeta/ExampleContainers/tree/main/Payload\_Type](https://github.com/MythicMeta/ExampleContainers/tree/main/Payload\_Type)

{% hint style="warning" %}
Docker doesn't allow you to have capital letters in your image names, and when Mythic builds these containers, it uses the container's name as part of the image name. So, you can't have capital letters in your agent/translation container names. That's why you'll see things like `service_wrapper` instead of `serviceWrapper`
{% endhint %}

## Turning a VM into a Translation Container

Just like with Payload Types, a Translation container doesn't have to be a Dockerized instance. To turn any VM into a translation container just follow the general flow at [#turning-a-vm-into-a-mythic-container](./#turning-a-vm-into-a-mythic-container "mention")
