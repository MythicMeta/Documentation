# C2 Server Utilities

## C2 OPSEC Checks

C2 Profiles can optionally provide some operational security checks before allowing a payload to be created. For example, you might want to prevent operators from using a known-bad named pipe name, or you might want to prevent them from using infrastructure that you know is burned.

### Where is it?

These checks all happen within a single function per C2 profile with a function called `opsec`:

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
async def opsec(self, inputMsg: C2OPSECMessage) -> C2OPSECMessageResponse:
        response = C2OPSECMessageResponse(Success=True)
        response.Message = "Not Implemented, passing by default"
        response.Message += f"\nInput: {json.dumps(inputMsg.to_json(), indent=4)}"
        return response
```

From the code snippet above, you can see that this function gets in a request with all of the parameter values for that C2 Profile that the user provided. You can then either return success or error with a message as to why it passed or why it failed. If you return the error case, then the payload won't be built.

## C2 Server Configuration Checks

C2 servers know the most about their configuration. You can pass in the configuration for an agent and check it against the server's configuration to make sure everything matches up or get additional insight into how to configure potential redirectors.

```python
async def config_check(self, inputMsg: C2ConfigCheckMessage) -> C2ConfigCheckMessageResponse:
        response = C2ConfigCheckMessageResponse(Success=True)
        response.Message = "Not Implemented"
        response.Message += f"\nInput: {json.dumps(inputMsg.to_json(), indent=4)}"
        return response
```

## C2 Server Redirect Rules

C2 servers know the most about how their configurations work. You can pass in an agent's configuration and get information about how to generate potential redirector rules so that only your agent's traffic makes it through.

```python
async def redirect_rules(self, inputMsg: C2GetRedirectorRulesMessage) -> C2GetRedirectorRulesMessageResponse:
        response = C2GetRedirectorRulesMessageResponse(Success=True)
        response.Message = "Not Implemented"
        response.Message += f"\nInput: {json.dumps(inputMsg.to_json(), indent=4)}"
        return response
```
