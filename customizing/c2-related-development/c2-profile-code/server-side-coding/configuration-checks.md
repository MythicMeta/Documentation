---
description: Check Agent's Configuration Before Generation
---

# Configuration Checks

## What is this?

Configuration checks are optional checks implemented by a C2 profile to alert the operator if they're generating an agent with a C2 configuration that doesn't match the current C2 docker services.&#x20;

## When does this happen?

This check occurs every time an agent is generated, and this output is added to the payload's `build_message`. Thus, an operator sees it when generating a payload, but it's always viewable again from the created payloads page.

## What does it look like?

The function sits in the same place as all of the other C2 RPC functionality within a C2 Profile and gets passed a dictionary with information about all of the parameters the user supplied for their agent. This function executes from within the `mythic_service.py` file though, so if you want to access your C2 Profile's configuration, you'll need to access it via `../c2_code/config.json`.&#x20;

Here's the example from the `http` profile that tries to be very verbose in checking for issues:

```python
# The config_check function is called when a payload is created as a check to see if the parameters supplied
#   to the agent match up with what's in the C2 profile
# The input for "request" is a dictionary of:
# {
#   "action": "config_check",
#   "parameters": {
#       "param_name": "param_value",
#       "param_name2: "param_value2",
#   }
# }
#
# This function should return one of two things:
#   For success: {"status": "success", "message": "your success message here" }
#   For error: {"status": "error", "error": "your error message here" }
async def config_check(request):
    try:
        with open("../c2_code/config.json") as f:
            config = json.load(f)
            possible_ports = []
            for inst in config["instances"]:
                possible_ports.append({"port": inst["port"], "use_ssl": inst["use_ssl"]})
                if str(inst["port"]) == str(request["parameters"]["callback_port"]):
                    if "https" in request["parameters"]["callback_host"] and not inst["use_ssl"]:
                        message = f"C2 Profile container is configured to NOT use SSL on port {inst['port']}, but the callback host for the agent is using https, {request['parameters']['callback_host']}.\n\n"
                        message += "This means there should be the following connectivity for success:\n"
                        message += f"Agent via SSL to {request['parameters']['callback_host']} on port {inst['port']}, then redirection to C2 Profile container WITHOUT SSL on port {inst['port']}"
                        return {"status": "error", "error": message}
                    elif "https" not in request["parameters"]["callback_host"] and inst["use_ssl"]:
                        message = f"C2 Profile container is configured to use SSL on port {inst['port']}, but the callback host for the agent is using http, {request['parameters']['callback_host']}.\n\n"
                        message += "This means there should be the following connectivity for success:\n"
                        message += f"Agent via NO SSL to {request['parameters']['callback_host']} on port {inst['port']}, then redirection to C2 Profile container WITH SSL on port {inst['port']}"
                        return {"status": "error", "error": message}
                    else:
                        message = f"C2 Profile container and agent configuration match port, {inst['port']}, and SSL expectations.\n"
                        return {"status": "success", "message": message}
            message = f"Failed to find port, {request['parameters']['callback_port']}, in C2 Profile configuration\n"
            message += f"This could indicate the use of a redirector, or a mismatch in expected connectivity.\n\n"
            message += f"This means there should be the following connectivity for success:\n"
            if "https" in request["parameters"]["callback_host"]:
                message += f"Agent via HTTPS on port {request['parameters']['callback_port']} to {request['parameters']['callback_host']} (should be a redirector).\n"
            else:
                message += f"Agent via HTTP on port {request['parameters']['callback_port']} to {request['parameters']['callback_host']} (should be a redirector).\n"
            if len(possible_ports) == 1:
                message += f"Redirector then forwards request to C2 Profile container on port, {possible_ports[0]['port']}, {'WITH SSL' if possible_ports[0]['use_ssl'] else 'WITHOUT SSL'}"
            else:
                message += f"Redirector then forwards request to C2 Profile container on one of the following ports: {json.dumps(possible_ports)}\n"
            if "https" in request["parameters"]["callback_host"]:
                message += f"\nAlternatively, this might mean that you want to do SSL but are not using SSL within your C2 Profile container.\n"
                message += f"To add SSL to your C2 profile:\n"
                message += f"\t1. Go to the C2 Profile page\n"
                message += f"\t2. Click configure for the http profile\n"
                message += f"\t3. Change 'use_ssl' to 'true' and make sure the port is {request['parameters']['callback_port']}\n"
                message += f"\t4. Click to stop the profile and then start it again\n"
            return {"status": "success", "message": message}
    except Exception as e:
        return {"status": "error", "error": str(e)}
```
