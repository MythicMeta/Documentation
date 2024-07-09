# 7. Redirect Rules

## What is it?

This is a function operators can manually invoke for a payload to ask the payload's C2 profiles to generate a set of redirection rules for that payload. Nothing in Mythic knows more about a specific C2 profile than the C2 profile itself, so it makes sense that a C2 profile should be able to generate its own redirection rules for a given payload.

These redirection rules are up to the C2 Profile creators, but can include things like Apache mod\_rewrite rules, Nginx configurations, and more.

## Where is it?

Operationally, users can invoke this function from the created payloads page with a dropdown menu for the payload they're interested in. Functionally, this code lives in the class definition of your C2 Profile.

## What does it look like?

This function gets passed the same sort of information that the opsec check and configuration check functions get; namely, information about all of the payload's supplied c2 profile parameter values. This function can also access the C2 Profile's current configuration.

The format of the function is as follows:



{% tabs %}
{% tab title="Python" %}
<pre class="language-python"><code class="lang-python">async def redirect_rules(request: C2ProfileBase.C2GetRedirectorRulesMessage) -> C2ProfileBase.C2GetRedirectorRulesMessageResponse:
    return C2ProfileBase.C2GetRedirectorRulesMessageResponse(
        Success=True,
        Message="some mod rewrite rules here"
<strong>    )
</strong></code></pre>
{% endtab %}

{% tab title="Golang" %}
```go
GetRedirectorRulesFunction func(message C2GetRedirectorRuleMessage) C2GetRedirectorRuleMessageResponse
```

```go
package c2structs

// C2_REDIRECTOR_RULES STRUCTS

type C2_GET_REDIRECTOR_RULE_STATUS = string

type C2GetRedirectorRuleMessage struct {
   C2Parameters
}

type C2GetRedirectorRuleMessageResponse struct {
   Success bool   `json:"success"`
   Error   string `json:"error"`
   Message string `json:"message"`
}
```
{% endtab %}
{% endtabs %}
