---
description: Token awareness and Token tasking
---

# Tokens

Mythic supports Windows tokens in two ways: tracking which tokens are viewable and tracking which tokens are usable by the callback. The difference here, besides how they show up in the Mythic interface, is what the agent can do with the tokens. The idea is that you can list out all tokens on a computer, but that doesn't mean that the agent has a handle to the token for use with tasking.&#x20;

### Tokens

As part of the normal responses sent back from the agent, there's an additional key you can supply, `tokens` that is a list of token objects that can be stored in the Mythic database. These will be viewable from the "Search" -> "Tokens" page, but are not leveraged as part of further tasking. If you're curious about the fields, they're pulled from the output of the NtObjectManager.

```
{ "action": "post_response",
    "responses": [
        {
            "task_id": "uuid here",
            "user_output": "got some tokens yo",
            "tokens": [
                {
                    "TokenId": 18947, // required
                    "description": "", //
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