# Credentials

## Where is it?

Credentials can be found from the "Operational Views" -> "Credentials" page on the top navigation bar.&#x20;

## How is it integrated?

As part of command output, credentials can be registered automatically if the agent parses out the material. Otherwise, users can also manually register credentials. There are a few pieces of information required:

![](<../.gitbook/assets/Screen Shot 2020-02-17 at 10.19.11 PM.png>)

* The type of credential - This is more for situational awareness right now, but in the future will help the flow of how to treat the credential before use.
* Account - the account this credential applies to
* Realm - the domain for the credential or a generic realm in case this is a credential for something else. If the account is a local account, the Domain is the name of the computer.
* Credential - the actual credential
* Comment - any comment you want to store about the credential

![](<../.gitbook/assets/Screen Shot 2020-02-17 at 10.21.06 PM.png>)

On this page you can also see the task that created credentials (which can be `Manual Entry` ), who added in the credential, and when it was added.

### Integration into issuing commands

Command parameters can hook into this for auto populating by selecting which data they're interested in and then further manipulating it if they want. There are a few different `Credential-X` parameter types that can be selected to provide either all of the fields (JSON style) or for specific components (Realm, Account, Credential, etc).

### Autopopulation

Tasks can register credentials with the server in their responses by following [Credentials](../customizing/hooking-features/credentials.md) format.&#x20;
