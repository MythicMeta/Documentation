---
description: >-
  This page has the various different ways the initial checkin can happen and
  the encryption schemes used.
---

# Action: Checkin

## Endpoint

All messages go to the `/api/v1.4/agent_message` endpoint. These messages can be:

* POST request
  * message content in body
  * message content in FIRST header value
  * message content in FIRST cookie value
*   GET request

    * message content in FIRST header value
    * message content in FIRST cookie value
    * message content in FIRST query parameter



### A note about UUIDs

You will see a bunch of UUIDs mentioned throughout this section. All UUIDs are UUIDv4 formatted UUIDs (36 characters in length) and formatted like:

```
b446b886-ab97-49b2-b240-969a75393c06
```

In general, the UUID concatenated with the encrypted message provides a way to give context to the encrypted message without requiring a lot of extra pieces and without having to do a bunch of nested base64 encodings. 99% of the time, your messages will use your callbackUUID in the outer message. The outer UUID gives Mythic information about how to decrypt or interpret the following encrypted blob. In general:

* **payloadUUID** as the outer UUID tells Mythic to look up that payload UUID, then look up the C2 profile associated with it, find a parameter called `AESPSK`, and use that as the key to decrypt the message
* **tempUUID** as the outer UUID tells Mythic that this is a staging process. So, look up the UUID in the staging database to see information about the blob, such as if it's an RSA encrypted blob or is part of a Diffie-Hellman key exchange
* **callbackUUID** as the outerUUID tells Mythic that this is a full callback with an established encryption key or in plaintext.&#x20;

However, when your payload first executes, it doesn't have a callbackUUID, it's just a payloadUUID. This is why you'll see clarifiers as to which UUID we're referring to when doing specific messages. The whole goal of the `checkin` process is to go from a payload (and thus payloadUUID) to a full callback (and thus callbackUUID), so at the end of staging and everything you'll end up with a new UUID that you'll use as the outer UUID.

{% hint style="info" %}
If your already existing callback sends a checkin message more than once, Mythic simply uses that information to _update_ information about the callback rather than trying to register a new callback.
{% endhint %}

## Plaintext Checkin

The plaintext checkin is useful for testing or when creating an agent for the first time. When creating payloads, if there's an AESPSK parameter for the C2 profile, then this value will be auto-populated with the operation's 32Byte AES256 key. However, if you're doing plaintext comms, then you need to clear this value when creating your payload. Mythic looks at that outer `PayloadUUID` and checks if there's an associated encryption key with it in the database. If there is, Mythic will automatically try to decrypt the rest of the message, which will fail. This checkin has the following format:

```
Base64( PayloadUUID + JSON({
    "action": "checkin", // required
    "ip": "127.0.0.1", // internal ip address - required
    "os": "macOS 10.15", // os version - required
    "user": "its-a-feature", // username of current user - required
    "host": "spooky.local", // hostname of the computer - required
    "pid": 4444, // pid of the current process - required
    "uuid": "payload uuid", //uuid of the payload - required
    
    "architecture": "x64", // platform arch - optional
    "domain": "test", // domain of the host - optional
    "integrity_level": 3, // integrity level of the process - optional
    "external_ip": "8.8.8.8", // external ip if known - optional
    "encryption_key": "base64 of key", // encryption key - optional
    "decryption_key": "base64 of key", // decryption key - optional
    })
)
```

* The JSON section is not encrypted in any way, it's all plaintext.&#x20;

The checkin has the following response:

```
Base64( PayloadUUID + JSON({
    "action": "checkin",
    "id": "UUID", // new UUID for the agent to use
    "status": "success"
    })
)
```

From here on, the agent messages use the new UUID instead of the payload UUID.

## Static Encryption Checkin

This method uses a static AES256 key for all communications. This can be different for each payload that's created, or the operation specific key can be used. To use the operation specific key, make sure that the C2 profile being used has a parameter called "AESPSK" - this will be auto populated on payload creation with the operation key. This key needs to be a 32Byte, base64 encoded value.

The message sent will be of the form:

```
Base64( PayloadUUID + AES256(
    JSON({
        "action": "checkin", // required
        "ip": "127.0.0.1", // internal ip address - required
        "os": "macOS 10.15", // os version - required
        "user": "its-a-feature", // username of current user - required
        "host": "spooky.local", // hostname of the computer - required
        "pid": 4444, // pid of the current process - required
        "uuid": "payload uuid", //uuid of the payload - required
        
        "architecture": "x64", // platform arch - optional
        "domain": "test", // domain of the host - optional
        "integrity_level": 3, // integrity level of the process - optional
        "external_ip": "8.8.8.8", // external ip if known - optional
        "encryption_key": "base64 of key", // encryption key - optional
        "decryption_key": "base64 of key", // decryption key - optional
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
This first message from Agent -> Mythic has the Payload UUID as the outer UUID _and_ the Payload UUID inside the checkin JSON message. Once the agent gets the reply with a callbackUUID, all future messages will have this callbackUUID as the outer UUID.&#x20;
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
* Diffie-Hellman key exchange
  * leveraged by the viper agent

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

where the AES key initially used is defined as the AESPSK value when generating the payload. Similar to the [Initial Checkin](initial-checkin.md#static-encryption-checkin) with a static key, this value will be pre-populated based on the Operation's AESPSK value, but can be swapped out when creating the payload to be any AES256 key.

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
        "ip": "127.0.0.1", // internal ip address - required
        "os": "macOS 10.15", // os version - required
        "user": "its-a-feature", // username of current user - required
        "host": "spooky.local", // hostname of the computer - required
        "pid": 4444, // pid of the current process - required
        "uuid": "payload uuid", //uuid of the payload - required
        
        "architecture": "x64", // platform arch - optional
        "domain": "test", // domain of the host - optional
        "integrity_level": 3, // integrity level of the process - optional
        "external_ip": "8.8.8.8", // external ip if known - optional
        "encryption_key": "base64 of key", // encryption key - optional
        "decryption_key": "base64 of key", // decryption key - optional
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

### EKE by using Diffie-Hellman

The agent starts running and generates a new Diffie-Hellman key-pair. The agent then sends the following message to Mythic:

```
Base64( PayloadUUID + AES256(
    JSON({
        "action": "staging_dh",
        "pub_key": "public value for DH",
        "session_id": "20char string", // unique session ID for this callback
        })
    )
)
```

where the AES key initially used is defined as the AESPSK value when generating the payload. Similar to the [Initial Checkin](initial-checkin.md#static-encryption-checkin) with a static key, this value will be pre-populated based on the Operation's AESPSK value, but can be swapped out when creating the payload to be any AES256 key.

This message causes the following response:

```
Base64( PayloadUUID + AES256(
    JSON({
        "action": "staging_dh",
        "uuid": "UUID", // new UUID for the next message
        "session_key": Base64( DHPubKey( new aes session key ) ),
        "session_id": "same 20 char string back"
        })
    )
)
```

The response is encrypted with the same AESPSK as used initially, but there's a session\_key value that is encrypted with the public Diffie-Hellman negotiated public key that was in the initial message. The response also includes a new staging UUID for the agent to use. This is _not_ the final UUID for the new callback, this is a temporary UUID to indicate that the next message will be encrypted with the new AES key.

The next message from the agent to Mythic is as follows:

```
Base64( tempUUID + AES256(
    JSON({
        "action": "checkin", // required
        "ip": "127.0.0.1", // internal ip address - required
        "os": "macOS 10.15", // os version - required
        "user": "its-a-feature", // username of current user - required
        "host": "spooky.local", // hostname of the computer - required
        "pid": 4444, // pid of the current process - required
        "uuid": "payload uuid", //uuid of the payload - required
        
        "architecture": "x64", // platform arch - optional
        "domain": "test", // domain of the host - optional
        "integrity_level": 3, // integrity level of the process - optional
        "external_ip": "8.8.8.8", // external ip if known - optional
        "encryption_key": "base64 of key", // encryption key - optional
        "decryption_key": "base64 of key", // decryption key - optional
        })
    )
)
```

This checkin data is the same as all the other methods of checking in, the key things here are that the tempUUID is the temp UUID specified in the other message, the inner uuid is the payload uuid, and the AES key used is the negotiated one. It's with this information that Mythic is able to track the new messages as belonging to the same staging sequence and confirm that all of the information was transmitted properly. The final response is as follows:

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

#### Diffie-Hellman Encryption Details

* key length: 540
* Generator: 2
* Group: 17
