---
description: >-
  This page has the various different ways the initial checkin can happen and
  the encryption schemes used.
---

# 2. Checkin

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

{% hint style="info" %}
In egress agent messages, you can opt for a 16 Byte big endian format for the UUID. If Mythic gets a message from an agent with this format of UUID, then it will respond with the same format for the UUID. However, currently for P2P messages Mythic doesn't track the format for the UUID of the agent, so these will get the standard 36 character long UUID String.&#x20;
{% endhint %}

## Plaintext Checkin

The plaintext checkin is useful for testing or when creating an agent for the first time. When creating payloads, you can generate encryption keys _per c2 profile_. To do so, the C2 Profile will have a parameter that has an attribute called `crypto_type=True`. This will then signal to Mythic to either generate a new per-payload AES256\_HMAC key or (if your agent is using a translation container) tell your agent's translation container to generate a new key. In the `http` profile for example, this is a `ChooseOne` option between `aes256_hmac` or `none`. If you're doing plaintext comms, then you need to set this value to `none` when creating your payload. Mythic looks at that outer `PayloadUUID` and checks if there's an associated encryption key with it in the database. If there is, Mythic will automatically try to decrypt the rest of the message, which will fail. This checkin has the following format:

```json
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

Here's an example checkin message message:

{% code overflow="wrap" fullWidth="false" %}
```
ODA4NDRkMTktOWJmYy00N2Y5LWI5YWYtYzZiOTE0NGMwZmRjeyJhY3Rpb24iOiJjaGVja2luIiwiaXBzIjpbIjE3Mi4xNi4xLjEiLCIxOTIuMTY4LjAuMTE4IiwiMTkyLjE2OC4yMjguMCIsIjE5Mi4xNjguNTMuMSIsIjE5OC4xOS4yNDkuMyIsImZkMDc6YjUxYTpjYzY2OjA6YTYxNzpkYjVlOmFiNzplOWYxIiwiZmQ1MzpkYTlmOjk4MWE6NWI0Mjo4YjA6MzNjOTplMGE1OjIyNTYiLCJmZTgwOjoxIiwiZmU4MDo6MTQ3ZDpkYWZmOmZlZWM6YjQ2NCIsImZlODA6OjE0N2Q6ZGFmZjpmZWVjOmI0NjUiLCJmZTgwOjoxNDdkOmRhZmY6ZmVlYzpiNDY2IiwiZmU4MDo6MTQ3ZDpkYWZmOmZlZWM6YjQ2NyIsImZlODA6OjIyOmQxYzk6MWMyZTo5Mjk3IiwiZmU4MDo6MzQ3ZDpkYWZmOmZlY2U6M2ExNyIsImZlODA6OjNjMmQ6ODZiYjo4ZDk5OjJjNjciLCJmZTgwOjo4ODU3OjJhZmY6ZmU2NToyNTExIiwiZmU4MDo6ODg1NzoyYWZmOmZlNjU6MjUxMSIsImZlODA6OmFlZGU6NDhmZjpmZTAwOjExMjIiLCJmZTgwOjpjZTgxOmIxYzpiZDJjOjY5ZSIsImZlODA6OmQxMDM6N2IyNDo2YzliOjhlMjIiXSwib3MiOiJWZXJzaW9uIDEzLjQgKEJ1aWxkIDIyRjY2KSIsInVzZXIiOiJpdHNhZmVhdHVyZSIsImhvc3QiOiJzcG9va3kubG9jYWwiLCJwaWQiOjY1ODYsInV1aWQiOiI4MDg0NGQxOS05YmZjLTQ3ZjktYjlhZi1jNmI5MTQ0YzBmZGMiLCJhcmNoaXRlY3R1cmUiOiJhbWQ2NCIsImRvbWFpbiI6IiIsImludGVncml0eV9sZXZlbCI6MiwiZXh0ZXJuYWxfaXAiOiIiLCJwcm9jZXNzX25hbWUiOiIvVXNlcnMvaXRzYWZlYXR1cmUvRG9jdW1lbnRzL015dGhpY0FnZW50cy9wb3NlaWRvbi9QYXlsb2FkX1R5cGUvcG9zZWlkb24vcG9zZWlkb24vYWdlbnRfY29kZS9wb3NlaWRvbl93ZWJzb2NrZXRfaHR0cC5iaW4ifQ==
```
{% endcode %}

The checkin has the following response:

```json
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

```json
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

Here's an example message with encryption key of `hfN9Nk29S8LsjrE9ffbT9KONue4uozk+/TVMyrxDvvM=` and message:

{% code overflow="wrap" fullWidth="false" %}
```
ODA4NDRkMTktOWJmYy00N2Y5LWI5YWYtYzZiOTE0NGMwZmRjnZ/FcM9jnfvzAv/RYFPAvkGH8+nWHAGqxcBXSlPvq8jbCRoZrVvSSZOxNwg15q3Etz9hEb7Qunv1Sm3/8SSzp+ne4fxFObunQWzHo+7tS68csvn/uxqhiyvD83KK66xtPyGzPFlK1ZXD+wxDbo2M3iSYPEp0m5w+rQhzm5aTA6Gk6p0KSXovYvnY3TsJtdgVPlY1cFt75UzTd0iIFU8hJ+KbhyMUjJujLA6++sVrXuFps2TbAi21Z5Hr/g3/S6HAk/RSedKyXEZ6Hbbgx3gESsHa/QuVjP9Lz+Y6H9I4DtgEunCHddvruJUPqYxFGT2m8WbGc6AH6+m2ucexym0yBUryuFWfsrW6QSfcGUaVb4DWrVHtqHcXctYRNb7pOf0T/P26pFt77fgii4j0RgzTGod9QDWhSfvte+ffUWjsWKyixUffjIffj45sgDS0tvtT2Rej8gFiIpAs9F/oOH/ps5pRQeflULd1eH0GKh5WUcDwsjUa89KeOcts44J+E5+7trQ3q2q9Uy8S96DM8Nr5QryokeCD7J0goKZQPdutVXzwIvI9RT7zCQpV8CrRTpQ63L9P9IhIpyT+TDvorQd0v/I/DGb6Ev/ZUAxbyAR0JLJGjYYv1NUno5Ru2Plv1wsn82YanVF1V2LE1ii6DC7jclrkgfKN9Qhli+hIiUwSJ3YvFTT1ybHf/Fyw4ZZ6PiOIZIWgcJmHUHx//1TNvlTrmABitRpwb75yuJ6ZfYnKv/BlrQtJ9nFveNeYKP/rL7uYwPq3RY9IJRK7DBOqy53qiiysRfhimraW//sXc6duBmASW0ijZ21HKaqdVr72PMIJpEWghIznzpzEVpJqYj0uR9K/bL5W6kfIP43dyDBzGAGd87VBIcUTsIJLWaOHGPVmO3OmmtIfW34ivsX1TElTVjyrmKneQ+OTWww0RbXZdE5swvucXqC8wTuwybgwQWVPCvrBTBlv3iXgkP4dOjbvr1YZS+HpdbT5OEhwIqnDCXIqItVYx9Hz5BdfcBFbXUXk0SIQzWQj9xw+olYYQMrxomNvjuGxBkOmhTJf6yUyRK1Mp8b992FPBzLVRexYFc5FZxrI8CJeS91R3C21gb3SZH4EdKk1S3mR40O427TGYG5Hcqzqz5n0M6+cWORxUp7LKT34kDwgzHQK1h5kEoaGvGB1QDtx8GLsbfk/BqBoV2oHGJP1HHbVgYMgBTrkYObXOKFW8WyaUWcB1p/dSmW5Ww==
```
{% endcode %}

The message response will be of the form:

```json
Base64( PayloadUUID + AES256(
    JSON({
        "action": "checkin",
        "id": "callbackUUID", // callback UUID for the agent to use
        "status": "success"
        })
    )
)
```

Here's that sample message's response:

{% code overflow="wrap" %}
```
ODA4NDRkMTktOWJmYy00N2Y5LWI5YWYtYzZiOTE0NGMwZmRjyHcKh56jliiv87ReJE7QqK8edpLcV5cfywt8Lg1jWJzPc8b37zB9/mliG1HKH0dyF/jZqiSzUfSWEjgfhKa3DoLUqJOvnbpOYYsL3GvfWrps3/HQhZogSjwXnQmTehbADhXrOqA4622YMFjJbpykxdq7kpufn+12GDidwNybOlbg9ej8D/PpZVVdqL2RdASe
```
{% endcode %}

From here on, the agent messages use the new UUID instead of the payload UUID.

{% hint style="info" %}
This first message from Agent -> Mythic has the Payload UUID as the outer UUID _and_ the Payload UUID inside the checkin JSON message. Once the agent gets the reply with a callbackUUID, all future messages will have this callbackUUID as the outer UUID.
{% endhint %}

With that same example from above, the agent gets back a response of success with a new callback UUID. From there on, since it's a static encryption, we'll see a get tasking message like the following:

{% code overflow="wrap" %}
```
ODIyYmZmMWItYmRhMC00YmNlLWE0ZDMtYTZiZGIxMWI4YTVm3F56rkDEESX1GBAOQy3yaGiiAQABGkGxY66lNP7JS1rie8e7KbFHXwICOj67vvXpo5cik/9LYBqfQ8Ce5E3eUF1mExFX3EOzgAJd6Ey4fR93LoUTeMQQQZ3+ZMCnphaaDVbvJXCuWgoTMr/wO17H1k4zoAaMi+PHk0BXaaNyHMc=
```
{% endcode %}

Notice how the outer UUID is different, but the encryption key is still the same.

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

```json
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

When it says "base64 of public RSA key" you can do one of two things:

* Base64 encode the entire PEM exported key (including the ---BEGIN and ---END blocks)
* Use the already base64 encoded data that's inbetween the ---BEGIN and ---END blocks

Here is an example of the first message using encryption key `hfN9Nk29S8LsjrE9ffbT9KONue4uozk+/TVMyrxDvvM=`:&#x20;

{% code overflow="wrap" %}
```
ODA4NDRkMTktOWJmYy00N2Y5LWI5YWYtYzZiOTE0NGMwZmRj8g4Anp52+vJpizSe8aymY4zNe2qz6xOMb1P69phayqfka57u2gdDBPzOKlkCEYjWlqIFr4Cpfa0krrXDTiaLLyT/wWKulFO8Z7h+/YqIi/6S8pW4hi+5Ht8543vJvlfuVMnK3YIL9ci/xJvkXoUPUI0Gb2fz2+AILD/+9mJrLx4OuJ/FAlVgSlfC4MOMJSOnOKX0D2Q2zsThJyfxzMs/sY9wUEOuYJMVZG5OZzupb7r7GPwZ0ZyeZrxDukR3r979E+2ZTSYWTDMv58PeyRUtLcaMhqPCZJTyDy4ZNJ04MxHbIQCYXsnlcybHczwMGUYw99/bqd1XVD9GKP5zmj3bP600+PbHg0G0N1qHhSrcagCQAIRka1ybSyYmlYILKYUwgmlVCmIT5ERmlXbJu9xqxzKCzfxYoBWpy6I72goDPpZDoK+LFsCIpQAJoRUA/u0KD61ujJCvr+gs/TRv9UIcd+AzR0r7m/ziawaoh6YdYJfPoJBEWi4eozNSaxrnQBOkCul3cOW/SZbZ/UVP84fThFlFLQdGiajmayoa0aLGDnKSh1l8pyX4Of1fajKX3XbY2bLALeU8Tw99E9daNSKhORqMAlmIrfvAHhDHs1vj3ZXj+rKl5We4JYSNSFOL9JzB5OlctV5bd+IuruFc3fLZVkdivjpGczz9iXh3p7Q3M5Xt6m+ZxUwuGa1otrJV55skF3Lns7p6owDw71weJH0h9JvvgoXOTtf1u9HI0ACBzHxThX+yMhmBBP0wU1Lngl6hF4o/1uwNk96fbAGLg0b9njziGC2OQ0D88kaqZ8jJ7C2XQyf4hetQCZCyYPSgtjMw1Yq1qRM0fHbU5cAkKvQmiJMeByHetctfDcs4SvnY28Tb1SfGCnxzxMJ+IQIbatKcQhwbhpq0iavsuG7NUVIPGBhB/8hw3PkkKDb3gqgoKuOD0y8zRK/+DrVbDT3DmzGrmJAkfFXqahjW/aaSNHmqdxxXoI/3Ft1FGocLYAj9bGclW4nzjarRpvtA8fUwMg/vX1RZqFVN15FTp8qsjzKsL8ld0aWlaGcRulfQr9oIKyC+P0EV3a0rMuBO2q2SuSWefyVRaMWCx0gY2Gtrm+bN3ddb+koyUsNdoI7lTY5HirQ3qG0unq28D6Clm8Cok91kMQEaGZ28pZvFZVs3iaLxiPxhmfj+UAQ95ncziJqGrbAiJgTVAmF0bUHAOSD2HORzVeGHxKgFsqSnJvK5B1NUCDIa1ok3sbGo8yg7tc/63pcUPBGcMRRQg3WBN8msj14fDoXAJg3MGG+qzomagdyRFQieMfFeOm1O8SU/a3U9uFwSqhwo4EsE5sIgKPTwN7OVEFbEzNA5tpr65lBxlzC4y1o9Juo25G5QXhuCuSN2frsu4QMlTnxi8P0HHed17hJjY8kaBG6Pm3h9HH098nxiIStZBWSYWQPSXy4AImrT+3vcovjLXPColbKd3M5wRog5WQ1j05O2NQGZwBFDktWioMqIDGCWECvHWgCvPiLmeeCwsWEncqnmrRCwLpI+DXxUVEX9oJFBZhlnfX2iaWeuDw==
```
{% endcode %}

This message causes the following response:

```json
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

Here's that sample response from our above sample message:

{% code overflow="wrap" %}
```
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
```
{% endcode %}

The response is encrypted with the same initial AESPSK value as before. However, the `session_key` value is encrypted with the public RSA key that was in the initial message and base64 encoded. The response also includes a new staging UUID for the agent to use. This is _not_ the final UUID for the new callback, this is a temporary UUID to indicate that the next message will be encrypted with the new AES key.

The next message from the agent to Mythic is as follows:

```json
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

With our new temp UUID, the agent sends the following:

{% code overflow="wrap" %}
```
MzAzMzY1M2UtMjlkOC00ZDRjLTkxMDgtNTZjMTUxZjc3OWQ34uGQ1yO25qWInocvzRCTRenTlUB7u1oRScx+09PeZZfUrJtdfiEeMD6Xz/kdKUsZjr9LWhFdFcu/AmHzAqH3LmIuSOxnMexmlGT9ngU7NuMvSdRlYvVcsIPYMLbRptletLttCBIu7LhDbuifYFRNQ21TBDkpVgYTXoUk5+JzzTesGdWAhOwLlWvijpKM4nrPLx0fZagEHH4SycHRUuHlei8T7F1YFPm8RhxbONMAd1ckjDnPm5kdUPx0JwpuP975MV4cuHdez+mR6C/JP4B9yeP9hhnHmSFKq7OghnHQQ39prPQ9WArSJ+N8UJ+XOiACpjYon2Qyf0FqhRDdoojrY4sCRrF3Khw9mry+5j5WlHubsICpfi52X9QQMAGzUNeuUve6jMKLQwSclb+IzJ2KKUHtA0qcsdvqyQ2mvXxicAn0OinnP6Vk7ktqsn35UQi3+uuPP0PWf53Iji25/mRCO9MbEa8WQ7epon4H4Erc1yw+Dvfb61BoasPbzspFFVtcuqRkeUYUiIHkR9uzVmSgUJqk/R4cRFico7nK+Q0E6gL1Qyk4P7yPLm3E98wkvoB1Y108r9tKyAFjfZ13MrZpZCsdt7y335hrymeZpt0V6/+ug3BIY1brcxE74fAnO4H5fUan7kPnvQmf/SsO4B9nHHRR/2pC1KYZF+vGw1My6alFPyyGZpzBnrsqyouFqhqOjS3iv8Yv3JY/IxpgJ/T4tXEs1MvgI19lufsBqX1PK9PiB04Y8+Igld6+6RTAwF+vf4utJTp4I/eeH8b0KZ9ABWzvjrPwj1nf1hN2Q0FU4YYBoXzKZ5kE8yZvYtfJpSqDGbsGW2gFr0nC2DybQ2QweLQ1RJAcBlU462jwP4h1ohuHRL2cynGqKaJa7XeILg5Da0ubMTg5fdEPoXMxaNYHdwbzV04WE7zKCru2T16AoncwWxzwcTqy6bRONdRPNFY72HlXmLgQ3R0a1J8VhlAt+7Z5I7rnz8S67rKL9xD1B4mJhhSeCj4k8y1/AgQ9i7ZIeLrcjsUcOo7Hw5Dl4QOBncuOn74DHWnZHgxUDQtB41GBwbJyeoi6ryHdMjUBEOK34f5msUh+HMDCHkLZi+4oM84K3L5oCQrPh1+b6FH5oXGO8pOXi2wHCAtzfF9LF5MfEAa5Lt6ZJpk9fzqZ6fPbFB0R/X/lKLyp6VaMGk5CBpLvwmyNGZnjSXra1r212sqIqFdGu9sVzMJ5daVLsjFPCg==
```
{% endcode %}

This checkin data is the same as all the other methods of checking in, the key things here are that the tempUUID is the temp UUID specified in the other message, the inner uuid is the payload UUID, and the AES key used is the negotiated one. It's with this information that Mythic is able to track the new messages as belonging to the same staging sequence and confirm that all of the information was transmitted properly. The final response is as follows:

```json
Base64( tempUUID + AES256(
    JSON({
        "action": "checkin",
        "id": "UUID", // new UUID for the agent to use
        "status": "success"
        })
    )
)
```

With our example, the agent gets back the following:

{% code overflow="wrap" %}
```
MzAzMzY1M2UtMjlkOC00ZDRjLTkxMDgtNTZjMTUxZjc3OWQ3Xgjq3vE9vduJliEd24jskrB+0gcqLc1ROCegwkvSjrqBLGFhrurNCsQnKIFYZ+YP6AGNjgzIlAXbLAPlsRAa6ge6BLQOsywskyHsE/2+65etgEH9plUzOdEv/nknwfdJKV7n7PHQsQ9w4nsV7j9DkeiuIQ+CnlBBRaPpCGYKo8m8keswNY7DssL1FE1t0DQ5
```
{% endcode %}

From here on, the agent messages use the new UUID instead of the payload UUID or temp UUID and continues to use the new negotiated AES key.

Lastly, here's an example after that exchange with the new callback UUID doing a get tasking request:

{% code overflow="wrap" %}
```
NTU4NWI2YzMtMmEzOC00ZGZlLWIwMDItNDI5ZjQ5Mzk4YzIxtpTh3cK5yOJ+RlbVJkeVLSRd8ExZbahaQoXg9AbW5SD+wdueD+tPhtB18kcJqy9s10qfsTx/8gMlcw5emRMVm+w9bnScW0BKARoldBlp+31La3/+HsqEKvYaEK9gGcBlEK7mDVqaJlYxgkwWRNGZs4i3eIHpKCc9Gyyz7dyaQUk=
```
{% endcode %}

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

This section requires you to have a [#translation-containers](../../translation-containers.md#translation-containers "mention") associated with your payload type. The agent sends your own custom message to Mythic:

```
Base64( payloadUUID + customMessage )
```

Mythic looks up the information for the payloadUUID and calls your translation container's `translate_from_c2_format` function. That function gets a dictionary of information like the following:

```json
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

```json
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
