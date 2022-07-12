# Translation Containers

### Translation Containers

If you want to have a different form of communication between Mythic and your agent than the specific JSON messages that Mythic uses, then you'll need a "translation container". These "translation containers" have similar structure to C2 Profiles because they're loosely related, but they are special containers that sit along with your Payload Type containers.

The first thing you'll need to do is specify the name of the container in your associated Payload Type class code. Update the Payload Type's class to include a line like `translation_container = "binaryTranslator"` after the supported\_os line. Now we need to create the container. This is pretty similar to the C2 Profile containers, only easier.

If we want our container to be called `binaryTranslator`, then copy the `Example_Translator` folder to the `C2_Profiles` folder with the name `binaryTranslator`. You'll notice that the contents of the `Example_Translator` are very similar to those of the Example C2 Profile code:

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

If a `translation_container` is specified for your Payload Type, then these two functions will be called as Mythic processes requests from your agent.

You then need to get the new container associated with the docker-compose file that Mythic uses, so run `sudo ./mythic-cli c2 add binaryTranslator`. Now you can start the container with `sudo ./mythic-cli c2 start binaryTranslator` and you should see the container pop up as a sub heading of your payload container.

Additionally, if you're leveraging a payload type that has `mythic_encrypts = False` and you're doing any cryptography, then you should use this same process and perform your encryption and decryption routines here. This is why Mythic provides you with the associated keys you generated for encryption, decryption, and which profile you're getting a message from.

### To Recap:

So, after copying that `Example_Translator` over to your `Mythic/C2_Profiles/` folder, you should:

1. change the name of the folder to be something else (like `my_translator`)
2. in your agent's builder.py file (where you define all the attributes of your payload type) add in the line `translation_container = "my_translator"`
3. If you want to handle encryption/decryption instead of Mythic, then in that same builder.py file set `mythic_encrypts = False`
4. Now that all the "pieces" are there, we need to get them all synced up. First we need to add that container to our docker file (`sudo ./mythic-cli c2 add my_translator`). Then we'll need to start it (`sudo ./mythic-cli c2 start my_translator`). That gets the container created and started.
5. The last piece is restarting your agent's container to sync in the new information that says it has a translation container, so `sudo ./mythic-cli payload start my_agent`)

{% hint style="warning" %}
Docker doesn't allow you to have capital letters in your image names, and when Mythic builds these containers, it uses the container's name as part of the image name. So, you can't have capital letters in your agent/translation container names. That's why you'll see things like `service_wrapper` instead of `serviceWrapper`
{% endhint %}

## Turning a VM into a Translation Container

Just like with Payload Types, a Translation container doesn't have to be a Dockerized instance. To turn any VM into a translation container, just do the following:

1. Install python 3.8+ in the VM or on the computer
2. `pip3 install mythic-translator-container` (this has all of the definitions and functions for the container to sync with Mythic and issue RPC commands). Make sure you get the right version of this PyPi package for the version of Mythic you're using ([#current-payloadtype-versions](container-syncing.md#current-payloadtype-versions "mention")).
3. Create a folder on the computer or VM (let's call it path `/pathA`). Essentially, your `/pathA` path will be the new `C2_Profiles/[translator name]` folder. From the Mythic install, copy the contents of `Mythic/Example_Translator` to `/pathA`. So, you should have  `/pathA/mythic/` and `/pathA/Dockerfile` (that last one won't matter for us though).
4. Edit the `/pathA/mythic/rabbitmq_config.json` with the parameters you need
   1. the `host` value should be the IP address of the main Mythic install
   2. the `name` value should be the name of the translation container (this is tied into how the routing is done within rabbitmq). For Mythic's normal docker containers, this is set to `hostname` because the hostname of the docker container is set to the name of the payload type. For this case though, that might not be true. So, you can set this value to the name of your translation type instead (this must match your translation name **exactly**).
   3. the `container_files_path` should be the absolute path to the folder in step 3 (`/pathA/`in this case).
   4. leave `virtual_host` and `username` the same
   5. You'll need the password of rabbitmq from your Mythic instance. You can either get this from the `Mythic/.env` file, by running `sudo ./mythic-cli config get rabbitmq_password`, or if you run `sudo ./mythic-cli config payload` you'll see it there too.
5. External agents need to connect to `mythic_rabbitmq` in order to send/receive messages. By default, this container is bound on localhost only. In order to have an external agent connect up, you will need to adjust this in the `Mythic/.env` file to have `RABBITMQ_BIND_LOCALHOST_ONLY=false` and restart Mythic (`sudo ./mythic-cli restart`). The `sudo ./mythic-cli config payload` will ask if you want to do this too.
6. Run `python3.8 mythic_service.py` and now you should see this container pop up in the UI
