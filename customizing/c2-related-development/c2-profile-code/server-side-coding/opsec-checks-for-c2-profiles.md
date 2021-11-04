# OPSEC Checks For C2 Profiles

## OPSEC scripting

When creating payloads, Mythic will send a C2 Profile's parameters to the associated C2 Profile container for an "opsec check". This is a function that you can choose to write (or not) to look over the C2-specific parameter values that an operator selected to see if they pass your risk tolerance. This function lives in the following path: `C2_Profiles/[c2 profile name]/mythic/c2_functions/C2_RPC_Functions.py`.&#x20;

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
    if request["parameters"]["callback_host"] == "https://domain.com":
        return {"status": "error", "error": "Callback Host is set to default of https://domain.com!\n"}
    if request["parameters"]["callback_host"].count(":") == 2:
        return {"status": "error", "error": f"Callback Host specifies a port ({request['parameters']['callback_host']})! This should be omitted and specified in the Callback Port parameter.\n"}
    if "https" in request["parameters"]["callback_host"] and request["parameters"]["callback_port"] not in ["443", "8443", "7443"]:
        return {"status": "success", "message": f"Mismatch in callback host: HTTPS specified, but port {request['parameters']['callback_port']}, is not standard HTTPS port\n"}
    return {"status": "success", "message": "Basic OPSEC Check Passed\n"}
```

The function gets an array of the parameters specified in the `request["parameters"]` element. In the end, the function is returning success or error for if the OPSEC check passed or not.&#x20;

## When is this executed?

OPSEC checks for C2 profiles are executed every time a Payload is created. This means when an operator does it through the UI, when somebody scripts it out, and when a payload is automatically generated as part of tasking (such as for lateral movement or spawning new callbacks).
