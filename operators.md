# Operators

Mythic is meant to be used by multiple operators working together to accomplish operations. That typically means there's a lead operator, multiple other operators, and potentially people that are just spectating. Let's see how operators come into play throughout Mythic.

## Passwords & Authentication

Every user has their own password for authenticating to Mythic. On initial startup, one account is created with a password specified via the `MYTHIC_ADMIN_PASSWORD` environment variable or by creating a `Mythic/.env` file with `MYTHIC_ADMIN_PASSWORD=passwordhere` entry. This account can then be used to provision other accounts (or they can be created via the [scripting](scripting/scripting-operators.md) ability). If the admin password isn't specified via the environment variable or via the `Mythic/.env` file, then a random password is used. This password is only used on initial setup to create the first user, after that this value is no longer used.

Every user's password must be at least 12 characters long. If somebody tries to log in with an unknown account, all operations will get a notification about it. Similarly, if a user fails to log into their account 10 times in a row, the account will lock. The only account that will not lock out is that initial account that's created. Instead, that account will throttle authentication attempts to 1 a minute.

## Operator Permissions

There are a few different kinds of operator permissions throughout Mythic.

* Admin - This is a global setting that grants users the ability to see all operations, unlock all callbacks, and interact with everything in Mythic. The only account that has this initially is the first account created and only Admin accounts can grant other admin accounts this level of permission. Similarly, only admin accounts can remove admin permissions from other admin accounts.
* Operation Admin - This is the lead of a specific operation. The operation admin can unlock anybody else's callback, can bypass any opsec check, and has full rights over that operation.
* Operator - This is the normal permissions for a user. They can be added to operations by Admins or by the Operation Admin, and within an operation, they can bypass opsec checks for operators and only unlock the callbacks they locked.
* Spectator - This account permission has no permissions to make any modifications within Mythic for their operation. They can still query and see all tasks/responses/artifacts/etc within an operation, but they cannot issue tasks, lock callbacks, create payloads, etc.
