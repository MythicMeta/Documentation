# Redirect Rules

## What is it?

This is a function operators can manually invoke for a payload to ask the payload's C2 profiles to generate a set of redirection rules for that payload. Nothing in Mythic knows more about a specific C2 profile than the C2 profile itself, so it makes sense that a C2 profile should be able to generate its own redirection rules for a given payload.

These redirection rules are up to the C2 Profile creators, but can include things like Apache mod\_rewrite rules, Nginx configurations, and more.

## Where is it?

Operationally, users can invoke this function from the created payloads page with a dropdown menu for the payload they're interested in. Functionally, this code lives in the same place for a C2 profile as all of the other RPC-based functions for a C2 profile.

## What does it look like?

This function gets passed the same sort of information that the opsec check and configuration check functions get; namely, information about all of the payload's supplied c2 profile parameter values. This function can also access the C2 Profile's current configuration by looking in the c2\_code folder (i.e. `../c2_code/config.json`).&#x20;

The format of the function is as follows:

```python
# The redirect_rules function is called on demand by an operator to generate redirection rules for a specific payload
# The input for "request" is a dictionary of:
# {
#   "action": "redirect_rules",
#   "parameters": {
#       "param_name": "param_value",
#       "param_name2: "param_value2",
#   }
# }
# This function should return one of two things:
#   For success: {"status": "success", "message": "your success message here" }
#   For error: {"status": "error", "error": "your error message here" }
async def redirect_rules(request):
    return {"status": "success", "message": "use the following mod_rewrite rules!"}
```
