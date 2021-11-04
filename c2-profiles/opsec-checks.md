# OPSEC Checks

## C2 OPSEC Checks

C2 Profiles can optionally provide some operational security checks before allowing a payload to be created. For example, you might want to prevent operators from using a known-bad named pipe name, or you might want to prevent them from using infrastructure that you know is burned.

### Where is it?

These checks all happen within a single function per C2 profile. For each one, they're always located at: `Mythic/C2_Profiles/[ProfileName]/mythic/c2_functions/C2_RPC_functions.py` with a function called `opsec`:

```python
# The opsec function is called when a payload is created as a check to see if the parameters supplied are good
# The input for "request" is a dictionary of:
# {
#   "action": "opsec",
#   "parameters": {
#       "param_name": "param_value",
#       "param_name2: "param_value2",
#   }
# }
# This function should return one of two things:
#   For success: {"status": "success", "message": "your success message here" }
#   For error: {"status": "error", "error": "your error message here" }
async def opsec(request):
    return {"status": "success", "message": "No OPSEC Check Performed"}
```

From the code snippet above, you can see that this function gets in a request with all of the parameter values for that C2 Profile that the user provided. You can then either return success or error with a message as to why it passed or why it failed. If you return the error case, then the payload won't be built.
