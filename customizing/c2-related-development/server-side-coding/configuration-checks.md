---
description: Check Agent's Configuration Before Generation
---

# 4. Configuration Checks

## What is this?

Configuration checks are optional checks implemented by a C2 profile to alert the operator if they're generating an agent with a C2 configuration that doesn't match the current C2 docker services.

## When does this happen?

This check occurs every time an agent is generated, and this output is added to the payload's `build_message`. Thus, an operator sees it when generating a payload, but it's always viewable again from the created payloads page.

## What does it look like?

The function is part of your C2 Profile's class definition, so it has access to your local `config.json` file as well as the instance configuration from the agent.



{% tabs %}
{% tab title="Python" %}
```python
async def config_check(request: C2ProfileBase.C2ConfigCheckMessage) -> C2ProfileBase.C2ConfigCheckMessageResponse:
    return C2ProfileBase.C2ConfigCheckMessageResponse(
        Success=True,
        Message="Some configuration check message",
        )
```
{% endtab %}

{% tab title="Golang" %}


```go
ConfigCheckFunction        func(message C2ConfigCheckMessage) C2ConfigCheckMessageResponse 
```



```go
package c2structs

// C2_CONFIG_CHECK STRUCTS

type C2ConfigCheckMessage struct {
   C2Parameters
}

type C2ConfigCheckMessageResponse struct {
   Success bool   `json:"success"`
   Error   string `json:"error"`
   Message string `json:"message"`
}
```
{% endtab %}
{% endtabs %}
