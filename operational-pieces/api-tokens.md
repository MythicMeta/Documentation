# API Tokens

## What are they?

API tokens are special JSON web tokens (JWTs) that Mythic can create per-user that don't expire automatically. This allows you to do long-term scripting capabilities without having to periodically check if your current access-token is expired, going through the refresh process, and then continuing along with whatever you were doing.

## Where are they?

They're located in your settings page (click your name in the top right and click settings).

## How are they used?

When making a request with an API token, set the `Header` of `apitoken` with a value of your API token. This is in contrast to normal JWT usage where the header is `Authorization` and the value is `Bearer: <token here>`.
