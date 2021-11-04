# Translation Containers

### Translation Containers

If you want to have a different form of communication between Mythic and your agent than the specific JSON messages that Mythic uses, then you'll need a "translation container".  These "translation containers" have similar structure to C2 Profiles because they're loosely related, but they are special containers that sit along with your Payload Type containers.&#x20;

The first thing you'll need to do is specify the name of the container in your associated Payload Type class code.  Update the Payload Type's class to include a line like `translation_container = "binaryTranslator"` after the supported\_os line. Now we need to create the container. This is pretty similar to the C2 Profile containers, only easier.

If we want our container to be called `binaryTranslator`, then copy the `Example_Translator` folder to the `Payload_Types` folder with the name `binaryTranslator`. You'll notice that the contents of the `Example_Translator` are very similar to those of the Example C2 Profile code:

![](<../../.gitbook/assets/Screen Shot 2020-12-10 at 3.56.36 PM.png>)

You only need to edit the `C2_RPC_functions.py` file. This contains three functions - one for translating from your custom c2 into Mythic JSON, one from converting from Mythic JSON to your custom c2, and one for generating custom crypto encryption/decryption .

```python
import json
import base64
import sys
# translate_from_c2_format gets a message from Mythic that is in the c2-specific format
#     and returns a message that's translated into Mythic's JSON format
# If the associated C2Profile has `mythic_encrypts` set to False, then this function should also decrypt
#     the message
# request will be JSON with the following format:
# { "action": "translate_from_c2_format",
#   "enc_key": None or base64 of key if Mythic knows of one,
#   "dec_key": None or base64 of key if Mythic knows of one,
#   "uuid": uuid of the message,
#   "profile": name of the c2 profile,
#   "mythic_encrypts": True or False if Mythic thinks Mythic does the encryption or not,
#   "type": None or a keyword for the type of encryption. currently only option besides None is "AES256"
#   "message": base64 of the message that's currently in c2 specific format
# }
# This should return the JSON of the message in Mythic format


async def translate_from_c2_format(request) -> dict:
    if not request["mythic_encrypts"]:
        return json.loads(base64.b64decode(request["message"]).decode()[36:])
    else:
        return json.loads(base64.b64decode(request["message"]))


# translate_to_c2_format gets a message from Mythic that is in Mythic's JSON format
#     and returns a message that's formatted into the c2-specific format
# If the associated C2Profile has `mythic_encrypts` set to False, then this function should also encrypt
#     the message
# request will be JSON with the following format:
# { "action": "translate_to_c2_format",
#   "enc_key": None or base64 of key if Mythic knows of one,
#   "dec_key": None or base64 of key if Mythic knows of one,
#   "uuid": uuid of the message,
#   "profile": name of the c2 profile,
#   "mythic_encrypts": True or False if Mythic thinks Mythic does the encryption or not,
#   "type": None or a keyword for the type of encryption. currently only option besides None is "AES256"
#   "message": JSON of the mythic message
# }
# This should return the bytes of the message in c2 specific format

async def translate_to_c2_format(request) -> bytes:
    if not request["mythic_encrypts"]:
        return base64.b64encode(request["uuid"].encode() + json.dumps(request["message"]).encode())
    else:
        return json.dumps(request["message"]).encode()


# generate_keys gets a message from Mythic that is in Mythic's JSON format
#     and returns a a JSON message with encryption and decryption keys for the specified type
# request will be JSON with the following format:
# { "action": "generate_keys",
#   "message": JSON of the C2 parameter that has a crypt_type that's not None and not empty
# }
# example:
# {"action":"generate_keys",
#   "message":{
#     "id":39,
#     "name":"AESPSK",
#     "default_value":"aes256_hmac\nnone",
#     "required":false,
#     "randomize":false,
#     "verifier_regex":"",
#     "parameter_type":"ChooseOne",
#     "description":"Crypto type",
#     "c2_profile":"http",
#     "value":"none",
#     "payload":"be8bd7fa-e095-4e69-87aa-a18ba73288cb",
#     "instance_name":null,
#     "operation":null,
#     "callback":null}}
# This should return the dictionary of keys like:
# {
#   "enc_key": "base64 of encryption key here",
#   "dec_key": "base64 of decryption key here",
# }

async def generate_keys(request) -> dict:
    return {"enc_key": None, "dec_key": None}

```

If a `translation_container` is specified for your Payload Type, then these two functions will be called as Mythic processes requests from your agent.&#x20;

You then need to get the new container associated with the docker-compose file that Mythic uses, so run `sudo ./mythic-cli payload add binaryTranslator`. Now you can start the container with `sudo ./mythic-cli payload start binaryTranslator` and you should see the container pop up as a sub heading of your payload container.

Additionally, if you're leveraging a payload type that has  `mythic_encrypts = False` and you're doing any cryptography, then you should use this same process and perform your encryption and decryption routines here. This is why Mythic provides you with the associated keys you generated for encryption, decryption, and which profile you're getting a message from.

### To Recap:

So, after copying that `Example_Translator` over to your `Mythic/Payload_Types/` folder, you should:

1. change the name of the folder to be something else (like `my_translator`)
2. in your agent's builder.py file (where you define all the attributes of your payload type) add in the line `translation_container = "my_translator"`
3. If you want to handle encryption/decryption instead of Mythic, then in that same builder.py file set `mythic_encrypts = False`
4. Now that all the "pieces" are there, we need to get them all synced up. First we need to add that container to our docker file (`sudo ./mythic-cli payload add my_translator`). Then we'll need to start it (`sudo ./mythic-cli payload start my_translator`). That gets the container created and started.
5. The last piece is restarting your agent's container to sync in the new information that says it has a translation container, so `sudo ./mythic-cli payload start my_agent`)

{% hint style="warning" %}
Docker doesn't allow you to have capital letters in your image names, and when Mythic builds these containers, it uses the container's name as part of the image name. So, you can't have capital letters in your agent/translation container names. That's why you'll see things like `service_wrapper` instead of `serviceWrapper`
{% endhint %}
