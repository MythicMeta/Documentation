---
description: Token awareness and Token tasking
---

# Tokens

Mythic supports Windows tokens in two ways: tracking which tokens are viewable and tracking which tokens are usable by the callback. The difference here, besides how they show up in the Mythic interface, is what the agent can do with the tokens. The idea is that you can list out all tokens on a computer, but that doesn't mean that the agent has a handle to the token for use with tasking.&#x20;

### Tokens

As part of the normal responses sent back from the agent, there's an additional key you can supply, `tokens` that is a list of token objects that can be stored in the Mythic database. These will be viewable from the "Search" -> "Tokens" page, but are not leveraged as part of further tasking. If you're curious about the fields, they're pulled from the output of the [NtObjectManager](https://github.com/googleprojectzero/sandbox-attacksurface-analysis-tools/blob/main/NtObjectManager/NtTokenFunctions.ps1).

```
{ "action": "post_response",
    "responses": [
        {
            "task_id": "uuid here",
            "user_output": "got some tokens yo",
            "tokens": [
                {
                    "TokenId": 18947, // required
                    "description": "", // optional
                    "User": "bob", // optional
                    "Groups": "", //optional
                    "EnabledGroups": "", //optional p.TextField(null=True)
                    "DenyOnlyGroups": "", //optional p.TextField(null=True)
                    "GroupCount": 4, //optional p.IntegerField(null=True)
                    # enum type, 0 is Primary 1 is Secondary
                    "TokenType": 0, //optional p.IntegerField(null=True)
                    "Owner": "", //optional p.TextField(null=True)
                    "PrimaryGroup": "", //optional p.TextField(null=True)
                    "DefaultDacl": "", //optional p.TextField(null=True)
                    "Source": "", //optional  p.TextField(null=True)
                    "RestrictedSids": "", //optional p.TextField(null=True)
                    "RestrictedSidsCount": "", //optional p.TextField(null=True)
                    # enum type
                    "ImpersonationLevel": 0, //optional p.IntegerField(null=True)
                    "SessionId": 0, //optional p.IntegerField(null=True)
                    "SandboxInert": false, //optional p.BooleanField(null=True)
                    "Origin": 0, //optional p.IntegerField(null=True)
                    "ElevationType": 0, //optional p.IntegerField(null=True)
                    "Elevated": false, //optional p.BooleanField(null=True)
                    "HasRestrictions": false, //optional = p.BooleanField(null=True)
                    "UIAccess": false, //optional = p.BooleanField(null=True)
                    "VirtualizationAllowed": false, //optional = p.BooleanField(null=True)
                    "VirtualizationEnabled": false, //optional = p.BooleanField(null=True)
                    "Restricted": false, //optional = p.BooleanField(null=True)
                    "WriteRestricted": false, //optional = p.BooleanField(null=True)
                    "Filtered": false, //optional = p.BooleanField(null=True)
                    "NotLow": true, //optional = p.BooleanField(null=True)
                    "Flags": "", //optional = p.TextField(null=True)
                    "NoChildProcess": false, //optional = p.BooleanField(null=True)
                    "Capabilities": "", //optional = p.TextField(null=True)
                    "MandatoryPolicy": "", //optional = p.TextField(null=True)
                    "LogonSid": "", //optional = p.TextField(null=True)
                    "IntegrityLevelSid": 0, //optional = p.IntegerField(null=True)
                    "AppContainerNumber": 0, //optional = p.IntegerField(null=True)
                    "IntegrityLevel": 1, //optional = p.IntegerField(null=True)
                    "SecurityAttributes": "", //optional = p.TextField(null=True)
                    "DeviceClaimAttributes": "", //optional = p.TextField(null=True)
                    "UserClaimAttributes": "", //optional = p.TextField(null=True)
                    "RestrictedUserClaimAttributes": "", //optional = p.TextField(null=True)
                    "RestrictedDeviceClaimAttributes": "", //optional = p.TextField(null=True)
                    "AppContainer": false, //optional = p.BooleanField(null=True)
                    "LowPrivilegeAppContainer": false, //optional = p.BooleanField(null=True)
                    "AppContainerSid": "", //optional = p.TextField(null=True)
                    "PackageName": "", //optional = p.TextField(null=True)
                    "DeviceGroups": "", //optional = p.TextField(null=True)
                    "RestrictedDeviceGroups": "", //optional = p.TextField(null=True)
                    "Privileges": "", //optional = p.TextField(null=True)
                    "FullPath": "", //optional = p.TextField(null=True)
                    "TrustLevel": "", //optional = p.TextField(null=True)
                    "IsPseudoToken": false, //optional = p.BooleanField(null=True)
                    "IsSandbox": false, //optional = p.BooleanField(null=True)
                    "PackageFullName": "", //optional = p.TextField(null=True)
                    "AppId": "", //optional = p.TextField(null=True)
                    "AppModelPolicies": "", //optional = p.TextField(null=True)
                    "AppModelPolicyDictionary": "", //optional = p.TextField(null=True)
                    "BnoIsolationPrefix": "", //optional = p.TextField(null=True)
                    "PackageIdentity": "", //optional = p.TextField(null=True)
                    "AuditPolicy": "", //optional = p.TextField(null=True)
                    "PrivateNamespace": false, //optional = p.BooleanField(null=True)
                    "IsRestricted": false, //optional = p.BooleanField(null=True)
                    "ProcessUniqueAttribute": "", //optional = p.TextField(null=True)
                    "GrantedAccess": "", //optional = p.TextField(null=True)
                    "GrantedAccessGeneric": "", //optional = p.TextField(null=True)
                    "GrantedAccessMask": "", //optional = p.IntegerField(null=True)
                    "SecurityDescriptor": "", //optional = p.TextField(null=True)
                    "Sddl": "", //optional = p.TextField(null=True)
                    "Handle": 12345, //optional = p.IntegerField(null=True)
                    "NtTypeName": "", //optional = p.TextField(null=True)
                    "NtType": 2, //optional = p.IntegerField(null=True)
                    "Name": "", //optional = p.TextField(null=True)
                    "CanSynchronize": false, //optional = p.BooleanField(null=True)
                    "AttributesFlags": 0, //optional = p.IntegerField(null=True)
                    "HandleReferenceCount": 1, //optional = p.IntegerField(null=True)
                    "PointerReferenceCount": 1, //optional = p.IntegerField(null=True)
                    "Inherit": false, //optional = p.BooleanField(null=True)
                    "ProtectFromClose": false, //optional = p.BooleanField(null=True)
                    "Address": 12234, //optional = p.IntegerField(null=True)
                    "IsContainer": false, //optional = p.BooleanField(null=True)
                    "IsClosed": false, //optional = p.BooleanField(null=True)
                    
                }
            ]
        }
    ]
}
```

### Callback Tokens

If you want to be able to leverage tokens as part of your tasking, you need to register those tokens with Mythic and the callback. This can be done as part of the normal `post_response` responses like everything else. The key here is to identify the right token - specifically via the unique combination of TokenID and host.

```
{"action": "post_response",
    "responses": [
        {
            "task_id": "uuid here",
            "output": "now tracking token 12345",
            "callback_tokens": [
                {
                    "action": "add", // could also be "remove"
                    "host": "a.b.com", //optional - default to callback host if not specified
                    "TokenId": 12345, // id 
                }
            ]
        }
    ]
}
                    
```

If the token `12345` hasn't been reported via the `tokens` key then it will be created and then associated with Mythic.

Once the token is created and associated with the callback, there will be a new dropdown menu  next to the tasking bar at the bottom of the screen where you can select to use the default token or one of the new ones specified. When you select a token to use in this way when issuing tasking, the `create_tasking` function's `task` object will have a new attribute, `task.token` that contains a dictionary of all the token's associated attributes. This information can then be used to send additional data with the task down to the agent to indicate which tokens should be used for the task as part of your parameters.&#x20;

Additionally, when getting tasks that have tokens associated with them, the `TokenId` value will be passed down to the agent as an additional field:\


```
{ "action": "get_tasking",
    "tasks": [
        {
            "command": "shell",
            "parameters": "whoami",
            "id": "uuid here",
            "timestamp": 1234567,
            "token": 12345
        }
    ]
}
```
