# Translation Containers

### Translation Containers

If you want to have a different form of communication between Mythic and your agent than the specific JSON messages that Mythic uses, then you'll need a "translation container".&#x20;

The first thing you'll need to do is specify the name of the container in your associated Payload Type class code. Update the Payload Type's class to include a line like `translation_container = "binaryTranslator"` . Now we need to create the container.&#x20;

The process for making a translation container is almost identical to a c2 profile or payload type container, we're simply going to change which classes we instantiate, but the rest of it is the same.

Unlike Payload Type and C2 Profile containers that mainly do everything over RabbitMQ for potentially long-running queues of jobs, Translation containers use gRPC for fast responses.&#x20;

If a `translation_container` is specified for your Payload Type, then the three functions defined in the following two examples will be called as Mythic processes requests from your agent.

You then need to get the new container associated with the docker-compose file that Mythic uses, so run `sudo ./mythic-cli add binaryTranslator`. Now you can start the container with `sudo ./mythic-cli start binaryTranslator` and you should see the container pop up as a sub heading of your payload container.

Additionally, if you're leveraging a payload type that has `mythic_encrypts = False` and you're doing any cryptography, then you should use this same process and perform your encryption and decryption routines here. This is why Mythic provides you with the associated keys you generated for encryption, decryption, and which profile you're getting a message from.

### Python

For the Python version, we simply instantiate our own subclass of the TranslationContainer class and provide three functions. In our `main.py` file, simply import the file with this definition and then start the service:&#x20;

```
mythic_container.mythic_service.start_and_run_forever()
```

```python
from mythic_container.TranslationBase import *


class myPythonTranslation(TranslationContainer):
    name = "binaryTranslator"
    description = "python translation service that doesn't change anything"
    author = "@its_a_feature_"

    async def generate_keys(self, inputMsg: TrGenerateEncryptionKeysMessage) -> TrGenerateEncryptionKeysMessageResponse:
        response = TrGenerateEncryptionKeysMessageResponse(Success=True)
        response.DecryptionKey = b""
        response.EncryptionKey = b""
        return response

    async def translate_to_c2_format(self, inputMsg: TrMythicC2ToCustomMessageFormatMessage) -> TrMythicC2ToCustomMessageFormatMessageResponse:
        response = TrMythicC2ToCustomMessageFormatMessageResponse(Success=True)
        response.Message = inputMsg.Message
        return response

    async def translate_from_c2_format(self, inputMsg: TrCustomMessageToMythicC2FormatMessage) -> TrCustomMessageToMythicC2FormatMessageResponse:
        response = TrCustomMessageToMythicC2FormatMessageResponse(Success=True)
        response.Message = inputMsg.Message
        return response
```

### GoLang

For the GoLang side of things, we instantiate an instance of the translationstructs.TranslationContainer struct with our same three functions. For GoLang though, we have an Initialize function to add this struct as a new definition to track.&#x20;

```go
import (
   "crypto/rand"
   "encoding/json"
   "errors"
   "github.com/MythicMeta/MythicContainer/logging"
   "github.com/MythicMeta/MythicContainer/translationstructs"
)

var myTranslationService = translationstructs.TranslationContainer{
   Name:        "myTranslationService",
   Description: "My test translation service that doesn't actually translate anything",
   Author:      "@its_a_feature_",
   GenerateEncryptionKeys: func(input translationstructs.TrGenerateEncryptionKeysMessage) translationstructs.TrGenerateEncryptionKeysMessageResponse {
      response := translationstructs.TrGenerateEncryptionKeysMessageResponse{
         Success: false,
      }
      if keys, err := GenerateKeysForPayload(input.CryptoParamValue); err != nil {
         logging.LogError(err, "Failed to determine valid crypto type")
         response.Error = err.Error()
         return response
      } else {
         response.Success = true
         response.EncryptionKey = keys.EncKey
         response.DecryptionKey = keys.DecKey
         return response
      }
   },
   TranslateMythicToCustomFormat: func(input translationstructs.TrMythicC2ToCustomMessageFormatMessage) translationstructs.TrMythicC2ToCustomMessageFormatMessageResponse {
      response := translationstructs.TrMythicC2ToCustomMessageFormatMessageResponse{}
      if input.MythicEncrypts {
         // mythic will take our resulting bytes and encrypt them, so just convert to bytes and return
         if jsonBytes, err := json.Marshal(input.Message); err != nil {
            response.Success = false
            response.Error = err.Error()
         } else {
            response.Success = true
            response.Message = jsonBytes
         }
      } else {

      }
      return response
   },
   TranslateCustomToMythicFormat: func(input translationstructs.TrCustomMessageToMythicC2FormatMessage) translationstructs.TrCustomMessageToMythicC2FormatMessageResponse {
      response := translationstructs.TrCustomMessageToMythicC2FormatMessageResponse{}
      outputMap := map[string]interface{}{}
      if input.MythicEncrypts {
         // mythic already decrypted these bytes, so just convert to map and return
         if err := json.Unmarshal(input.Message, &outputMap); err != nil {
            response.Success = false
            response.Error = err.Error()
         } else {
            response.Success = true
            response.Message = outputMap
         }
      } else {
         // we're expected to decrypt the bytes first, then convert them
      }

      return response
   },
}

func GenerateKeysForPayload(cryptoType string) (translationstructs.CryptoKeys, error) {
   switch cryptoType {
   case "aes256_hmac":
      bytes := make([]byte, 32)
      if _, err := rand.Read(bytes); err != nil {
         logging.LogError(err, "Failed to generate new random 32 bytes for aes256 key")
         return translationstructs.CryptoKeys{
            EncKey: nil,
            DecKey: nil,
            Value:  cryptoType,
         }, err
      }
      return translationstructs.CryptoKeys{
         EncKey: &bytes,
         DecKey: &bytes,
         Value:  cryptoType,
      }, nil
   case "none":
      return translationstructs.CryptoKeys{
         EncKey: nil,
         DecKey: nil,
         Value:  cryptoType,
      }, nil
   default:
      return translationstructs.CryptoKeys{
         EncKey: nil,
         DecKey: nil,
         Value:  cryptoType,
      }, errors.New("Unknown crypto type")
   }
}

func Initialize() {
   translationstructs.AllTranslationData.Get("myTranslationService").AddPayloadDefinition(myTranslationService)
}
```

Then, in our `main.go` code, we call the Initialize function and start the services:

```go
mytranslatorfunctions.Initialize()
// sync over definitions and listen
MythicContainer.StartAndRunForever([]MythicContainer.MythicServices{
   MythicContainer.MythicServiceTranslationContainer,
})
```

### Examples:

These examples can be found at the MythicMeta organization on GitHub:

{% embed url="https://github.com/MythicMeta/ExampleContainers/tree/main/Payload_Type" %}

{% hint style="warning" %}
Docker doesn't allow you to have capital letters in your image names, and when Mythic builds these containers, it uses the container's name as part of the image name. So, you can't have capital letters in your agent/translation container names. That's why you'll see things like `service_wrapper` instead of `serviceWrapper`
{% endhint %}

## Turning a VM into a Translation Container

Just like with Payload Types, a Translation container doesn't have to be a Dockerized instance. To turn any VM into a translation container just follow the general flow at [#turning-a-vm-into-a-mythic-container](./#turning-a-vm-into-a-mythic-container "mention")
