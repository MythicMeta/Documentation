# OPSEC Checks For C2 Profiles

## OPSEC scripting

When creating payloads, Mythic will send a C2 Profile's parameters to the associated C2 Profile container for an "opsec check". This is a function that you can choose to write (or not) to look over the C2-specific parameter values that an operator selected to see if they pass your risk tolerance. This function is part of your C2 Profile's class definition:



{% tabs %}
{% tab title="Python" %}
```python
async def opsec(self, request: C2ProfileBase.C2OPSECMessage):
    response = C2ProfileBase.C2OPSECMessageResponse(Success=True)
    return response
```


{% endtab %}

{% tab title="Golang" %}


```go
OPSECCheckFunction         func(message C2OPSECMessage) C2OPSECMessageResponse 
```



```go
package c2structs

// C2_OPSEC_CHECKS STRUCTS

type C2OPSECMessage struct {
   C2Parameters
}

type C2OPSECMessageResponse struct {
   Success bool   `json:"success"`
   Error   string `json:"error"`
   Message string `json:"message"`
}
```
{% endtab %}
{% endtabs %}

&#x20;In the end, the function is returning success or error for if the OPSEC check passed or not.

## When is this executed?

OPSEC checks for C2 profiles are executed every time a Payload is created. This means when an operator does it through the UI, when somebody scripts it out, and when a payload is automatically generated as part of tasking (such as for lateral movement or spawning new callbacks).
