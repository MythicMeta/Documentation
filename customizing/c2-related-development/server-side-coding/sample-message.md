# Sample Message



## What is it?

It's often useful to test your C2 redirector setup before your final deployment. It's also tough to know if there's an issue, if it could be with the agent, with a redirector, or with the C2 profile itself. Because of this, it can be very helpful for a C2 profile to generate a "sample message" that should fit all of the criteria based on an agent's configuration that you can either test configurations or even include in a report about how the C2 configuration works.



## Where is it?

On the created payloads page, there's an actions dropdown button next to each payload. That dropdown will contain an option to generate a sample message. This request takes that agent's configuration and forwards it along to the C2 profile.



## What does it look like?



{% tabs %}
{% tab title="Python" %}


```python
async def sample_message(self, inputMsg: C2SampleMessageMessage) -> C2SampleMessageMessageResponse:
    """Generate a sample message for this c2 profile based on the configuration specified

    :param inputMsg: Payload's C2 Profile configuration
    :return: C2SampleMessageMessageResponse detailing a sample message
    """
    response = C2SampleMessageMessageResponse(Success=True)
    response.Message = "Not Implemented"
    response.Message += f"\nInput: {json.dumps(inputMsg.to_json(), indent=4)}"
    return response
```
{% endtab %}

{% tab title="Golang" %}


```go
SampleMessageFunction      func(message C2SampleMessageMessage) C2SampleMessageResponse
```



```go
package c2structs

// C2_SAMPLE_MESSAGE STRUCTS

// C2SampleMessageMessage - Generate sample C2 Traffic based on this configuration so that the
// operator and developer can more easily troubleshoot
type C2SampleMessageMessage struct {
   C2Parameters
}

// C2SampleMessageResponse - Provide a string representation of the C2 Traffic that the corresponding
// C2SampleMessageMessage configuration would generate
type C2SampleMessageResponse struct {
   Success bool   `json:"success"`
   Error   string `json:"error"`
   Message string `json:"message"`
}
```
{% endtab %}
{% endtabs %}

