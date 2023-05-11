# OPSEC Checks For C2 Profiles

## OPSEC scripting

When creating payloads, Mythic will send a C2 Profile's parameters to the associated C2 Profile container for an "opsec check". This is a function that you can choose to write (or not) to look over the C2-specific parameter values that an operator selected to see if they pass your risk tolerance. This function is part of your C2 Profile's class definition:

```python
async def opsec(self, request: C2ProfileBase.C2OPSECMessage):
    response = C2ProfileBase.C2OPSECMessageResponse(Success=True)
    return response
```

&#x20;In the end, the function is returning success or error for if the OPSEC check passed or not.

## When is this executed?

OPSEC checks for C2 profiles are executed every time a Payload is created. This means when an operator does it through the UI, when somebody scripts it out, and when a payload is automatically generated as part of tasking (such as for lateral movement or spawning new callbacks).
