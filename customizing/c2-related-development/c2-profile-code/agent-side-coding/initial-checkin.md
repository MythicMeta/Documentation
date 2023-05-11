---
description: >-
  This page has the various different ways the initial checkin can happen and
  the encryption schemes used.
---

# Action: Checkin

### A note about UUIDs

You will see a bunch of UUIDs mentioned throughout this section. All UUIDs are UUIDv4 formatted UUIDs (36 characters in length) and formatted like:

```
b446b886-ab97-49b2-b240-969a75393c06
```

In general, the UUID concatenated with the encrypted message provides a way to give context to the encrypted message without requiring a lot of extra pieces and without having to do a bunch of nested base64 encodings. 99% of the time, your messages will use your callbackUUID in the outer message. The outer UUID gives Mythic information about how to decrypt or interpret the following encrypted blob. In general:

* **payloadUUID** as the outer UUID tells Mythic to look up that payload UUID, then look up the C2 profile associated with it, find a parameter called `AESPSK`, and use that as the key to decrypt the message
* **tempUUID** as the outer UUID tells Mythic that this is a staging process. So, look up the UUID in the staging database to see information about the blob, such as if it's an RSA encrypted blob or is part of a Diffie-Hellman key exchange
* **callbackUUID** as the outerUUID tells Mythic that this is a full callback with an established encryption key or in plaintext.

However, when your payload first executes, it doesn't have a callbackUUID, it's just a payloadUUID. This is why you'll see clarifiers as to which UUID we're referring to when doing specific messages. The whole goal of the `checkin` process is to go from a payload (and thus payloadUUID) to a full callback (and thus callbackUUID), so at the end of staging and everything you'll end up with a new UUID that you'll use as the outer UUID.

{% hint style="info" %}
If your already existing callback sends a checkin message more than once, Mythic simply uses that information to _update_ information about the callback rather than trying to register a new callback.
{% endhint %}

## Plaintext Checkin

The plaintext checkin is useful for testing or when creating an agent for the first time. When creating payloads, you can generate encryption keys _per c2 profile_. To do so, the C2 Profile will have a parameter that has an attribute called `crypto_type=True`. This will then signal to Mythic to either generate a new per-payload AES256\_HMAC key or (if your agent is using a translation container) tell your agent's translation container to generate a new key. In the `http` profile for example, this is a `ChooseOne` option between `aes256_hmac` or `none`. If you're doing plaintext comms, then you need to set this value to `none` when creating your payload. Mythic looks at that outer `PayloadUUID` and checks if there's an associated encryption key with it in the database. If there is, Mythic will automatically try to decrypt the rest of the message, which will fail. This checkin has the following format:

```
Base64( PayloadUUID + JSON({
    "action": "checkin", // required
    "uuid": "payload uuid", //uuid of the payload - required
    
    
    "ips": ["127.0.0.1"], // internal ip addresses - optional
    "os": "macOS 10.15", // os version - optional
    "user": "its-a-feature", // username of current user - optional
    "host": "spooky.local", // hostname of the computer - optional
    "pid": 4444, // pid of the current process - optional
    "architecture": "x64", // platform arch - optional
    "domain": "test", // domain of the host - optional
    "integrity_level": 3, // integrity level of the process - optional
    "external_ip": "8.8.8.8", // external ip if known - optional
    "encryption_key": "base64 of key", // encryption key - optional
    "decryption_key": "base64 of key", // decryption key - optional
    "process_name": "osascript", // name of the current process - optional
    })
)
```

{% hint style="info" %}
`integrity_level` is an integer from 1-4 that indicates the integrity level of the callback. On Windows, these levels correspond to low integrity (1) , medium integrity (2), high integrity (3), or SYSTEM integrity (4). On Linux, these don't have a great mapping, but you can think of (2) as a standard user, (3) as a user that's in the sudoers file or is able to run sudo, and (4) as the root user.
{% endhint %}

* The JSON section is not encrypted in any way, it's all plaintext.

The checkin has the following response:

```
Base64( PayloadUUID + JSON({
    "action": "checkin",
    "id": "UUID", // new UUID for the agent to use
    "status": "success"
    })
)
```

From here on, the agent messages use the new UUID instead of the payload UUID. This allows Mythic to track a payload trying to make a new callback vs a callback based on a payload.

## Static Encryption Checkin

This method uses a static AES256 key for all communications. This will be different for each payload that's created. When creating payloads, you can generate encryption keys _per c2 profile_. To do so, the C2 Profile will have a parameter that has an attribute called `crypto_type=True`. This will then signal to Mythic to either generate a new per-payload AES256\_HMAC key or (if your agent is using a translation container) tell your agent's translation container to generate a new key. In the `http` profile for example, this is a `ChooseOne` option between `aes256_hmac` or `none`. The key passed down to your agent during build time will be the base64 encoded version of the 32Byte key.

The message sent will be of the form:

```
Base64( PayloadUUID + AES256(
    JSON({
        "action": "checkin", // required
        "uuid": "payload uuid", //uuid of the payload - required
        
        
        "ips": ["127.0.0.1"], // internal ip addresses - optional
        "os": "macOS 10.15", // os version - optional
        "user": "its-a-feature", // username of current user - optional
        "host": "spooky.local", // hostname of the computer - optional
        "pid": 4444, // pid of the current process - optional
        "architecture": "x64", // platform arch - optional
        "domain": "test", // domain of the host - optional
        "integrity_level": 3, // integrity level of the process - optional
        "external_ip": "8.8.8.8", // external ip if known - optional
        "encryption_key": "base64 of key", // encryption key - optional
        "decryption_key": "base64 of key", // decryption key - optional
        "process_name": "osascript", // name of the current process - optional
        })
    )
)
```

The message response will be of the form:

```
Base64( PayloadUUID + AES256(
    JSON({
        "action": "checkin",
        "id": "callbackUUID", // callback UUID for the agent to use
        "status": "success"
        })
    )
)
```

From here on, the agent messages use the new UUID instead of the payload UUID.

{% hint style="info" %}
This first message from Agent -> Mythic has the Payload UUID as the outer UUID _and_ the Payload UUID inside the checkin JSON message. Once the agent gets the reply with a callbackUUID, all future messages will have this callbackUUID as the outer UUID.
{% endhint %}

#### AES256 Encryption Details

* Padding: PKCS7, block size of 16
* Mode: CBC
* IV is 16 random bytes
* Final message: IV + Ciphertext + HMAC
  * where HMAC is SHA256 with the same AES key over (IV + Ciphertext)

## Encrypted Key Exchange Checkins

There are two currently supported options for doing an encrypted key exchange in Mythic:

* Client-side generated RSA keys
  * leveraged by the apfell-jxa and poseidon agents
* Agent specific custom EKE

### EKE by generating client-side RSA keys

The agent starts running and generates a new 4096 bit Pub/Priv RSA key pair in memory. The agent then sends the following message to Mythic:

```
Base64( PayloadUUID + AES256(
    JSON({
        "action": "staging_rsa",
        "pub_key": "base64 of public RSA key",
        "session_id": "20char string", // unique session ID for this callback
        })
    )
)
```

where the AES key initially used is defined as the initial encryption value when generating the payload. When creating payloads, you can generate encryption keys _per c2 profile_. To do so, the C2 Profile will have a parameter that has an attribute called `crypto_type=True`. This will then signal to Mythic to either generate a new per-payload AES256\_HMAC key or (if your agent is using a translation container) tell your agent's translation container to generate a new key. In the `http` profile for example, this is a `ChooseOne` option between `aes256_hmac` or `none`.

This message causes the following response:

```
Base64( PayloadUUID + AES256(
    JSON({
        "action": "staging_rsa",
        "uuid": "UUID", // new UUID for the next message
        "session_key": Base64( RSAPub( new aes session key ) ),
        "session_id": "same 20 char string back"
        })
    )
)
```

The response is encrypted with the same initial AESPSK value as before. However, the `session_key` value is encrypted with the public RSA key that was in the initial message and base64 encoded. The response also includes a new staging UUID for the agent to use. This is _not_ the final UUID for the new callback, this is a temporary UUID to indicate that the next message will be encrypted with the new AES key.

The next message from the agent to Mythic is as follows:

```
Base64( tempUUID + AES256(
    JSON({
        "action": "checkin", // required
        "uuid": "payload uuid", //uuid of the payload - required
        
        
        "ips": ["127.0.0.1"], // internal ip addresses - optional
        "os": "macOS 10.15", // os version - optional
        "user": "its-a-feature", // username of current user - optional
        "host": "spooky.local", // hostname of the computer - optional
        "pid": 4444, // pid of the current process - optional
        "architecture": "x64", // platform arch - optional
        "domain": "test", // domain of the host - optional
        "integrity_level": 3, // integrity level of the process - optional
        "external_ip": "8.8.8.8", // external ip if known - optional
        "encryption_key": "base64 of key", // encryption key - optional
        "decryption_key": "base64 of key", // decryption key - optional
        "process_name": "osascript", // name of the current process - optional
        })
    )
)
```

This checkin data is the same as all the other methods of checking in, the key things here are that the tempUUID is the temp UUID specified in the other message, the inner uuid is the payload UUID, and the AES key used is the negotiated one. It's with this information that Mythic is able to track the new messages as belonging to the same staging sequence and confirm that all of the information was transmitted properly. The final response is as follows:

```
Base64( tempUUID + AES256(
    JSON({
        "action": "checkin",
        "id": "UUID", // new UUID for the agent to use
        "status": "success"
        })
    )
)
```

From here on, the agent messages use the new UUID instead of the payload UUID or temp UUID and continues to use the new negotiated AES key.

#### AES256 Encryption Details

* Padding: PKCS7, block size of 16
* Mode: CBC
* IV is 16 random bytes
* Final message: IV + Ciphertext + HMAC
  * where HMAC is SHA256 with the same AES key over (IV + Ciphertext)

#### RSA Encryption Details

* PKCS1\_OAEP
  * This is specifically OAEP with SHA1
* 4096Bits in size

### Your Own Custom EKE

This section requires you to have a [#translation-containers](../../../payload-type-development/translation-containers.md#translation-containers "mention") associated with your payload type. The agent sends your own custom message to Mythic:

```
Base64( payloadUUID + customMessage )
```

Mythic looks up the information for the payloadUUID and calls your translation container's `translate_from_c2_format` function. That function gets a dictionary of information like the following:

```
{
    "enc_key": None or base64 of key if Mythic knows of one,
    "dec_key": None or base64 of key if Mythic knows of one,
    "uuid": uuid of the message,
    "profile": name of the c2 profile,
    "mythic_encrypts": True or False if Mythic thinks Mythic does the encryption or not,
    "type": None or a keyword for the type of encryption. currently only option besides None is "AES256"
    "message": base64 of the message that's currently in c2 specific format
}
```

To get the `enc_key`, `dec_key`, and `type`, Mythic uses the payloadUUID to then look up information about the payload. It uses the `profile` associated with the message to look up the C2 Profile parameters and look for any parameter with a `crypto_type` set to `true`. Mythic pulls this information and forwards it all to your `translate_from_c2_format` function.

Ok, so that message gets your payloadUUID/crypto information and forwards it to your translation container, but then what?&#x20;

Normally, when the `translate_to_c2_format` function is called, you just translate from your own custom format to the standard JSON dictionary format that Mythic uses. No big deal. However, we're doing EKE here, so we need to do something a little different. Instead of sending back an action of `checkin`, `get_tasking`, `post_response`, etc, we're going to generate an action of `staging_translation`.

Mythic is able to do staging and EKE because it can save temporary pieces of information between agent messages. Mythic allows you to do this too if you generate a response like the following:

```
{
    "action": "staging_translation",
    "session_id": "some string session id you want to save",
    "enc_key": the bytes of an encryption key for the next message,
    "dec_key": the bytes of a decryption key for the next message,
    "crypto_type": "what type of crypto you're doing",
    "next_uuid": "the next UUID that'll be in front of the message",
    "message": "the raw bytes of the message that'll go back to your agent"
}
```

Let's break down these pieces a bit:

* `action` - this must be "staging\_translation". This is what indicates to Mythic once the message comes back from the `translate_from_c2_format` function that this message is part of staging.
* `session_id` - this is some random character string you generate so that we can differentiate between multiple instances of the same payload trying to go through the EKE process at the same time.
* `enc_key` / `dec_key` - this is the raw bytes of the encryption/decryption keys you want for the next message. The next time you get the `translate_from_c2_format` message for this instance of the payload going through staging, THESE are the keys you'll be provided.
* `crypto_type` - this is more for you than anything, but gives you insight into what the `enc_key` and `dec_key` are. For example, with the `http` profile and the `staging_rsa`, the crypto type is set to `aes256_hmac` so that I know exactly what it is. If you're handling multiple kinds of encryption or staging, this is a helpful way to make sure you're able to keep track of everything.
* `next_uuid` - this is the next UUID that appears in front of your message (instead of the payloadUUID). This is how Mythic will be able to look up this staging information and provide it to you as part of the next `translate_from_c2_format` function call.
* `message` - this is the actual raw bytes of the message you want to send back to your agent.

This process just repeats as many times as you want until you finally return from `translate_from_c2_format` an actual `checkin` message.

What if there's other information you need/want to store though? There are three RPC endpoints you can hit that allow you to store arbitrary data as part of your build process, translation process, or custom c2 process:\


* `create_agentstorage` - this take a unique\_id string value and the raw bytes data value. The `unique_id` is something that you need to generate, but since you're in control of it, you can make sure it's what you need. This returns a dictionary:
  * {"unique\_id": "your unique id", "data": "base64 of the data you supplied"}
* `get_agentstorage` - this takes the unique\_id string value and returns a dictionary of the stored item:&#x20;
  * {"unique\_id": "your unique id", "data": "base64 of the data you supplied"}
* `delete_agentstorage` - this takes the unique\_id string value and removes the entry from the database
