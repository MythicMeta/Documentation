# Auth

## Auth Structure

Auth containers allow you to extend Mythic's login capabilities by offloading the check from Mythic's salted + hashed database password to something else. This can either be towards an Identity Provider (IDP) like Microsoft ADFS (SSO), or towards a non-identity provider/SSO check (like attempted auth to an LDAP service).

{% tabs %}
{% tab title="Python" %}

{% endtab %}

{% tab title="Golang" %}
```go
type AuthDefinition struct {
	Name                  string   `json:"name"`
	Description           string   `json:"description"`
	IDPServices           []string `json:"idp_services"`
	NonIDPServices        []string `json:"non_idp_services"`
	GetIDPMetadata        func(GetIDPMetadataMessage) GetIDPMetadataMessageResponse
	GetIDPRedirect        func(GetIDPRedirectMessage) GetIDPRedirectMessageResponse
	ProcessIDPResponse    func(ProcessIDPResponseMessage) ProcessIDPResponseMessageResponse
	GetNonIDPMetadata     func(GetNonIDPMetadataMessage) GetNonIDPMetadataMessageResponse
	GetNonIDPRedirect     func(GetNonIDPRedirectMessage) GetNonIDPRedirectMessageResponse
	ProcessNonIDPResponse func(ProcessNonIDPResponseMessage) ProcessNonIDPResponseMessageResponse
	// Subscriptions - don't bother here, this will be auto filled out on syncing
	Subscriptions            []string                                                                                  `json:"subscriptions"`
	OnContainerStartFunction func(sharedStructs.ContainerOnStartMessage) sharedStructs.ContainerOnStartMessageResponse `json:"-"`
}
```

for example:

```go
func Initialize() {
    authName := "MyAuthProvider"
    myAuth := authstructs.AuthDefinition{
       Name:           authName,
       Description:    "A custom SSO auth provider for ADFS",
       IDPServices:    []string{"ADFS"},
       NonIDPServices: []string{"LDAP"},
       OnContainerStartFunction: func(message sharedStructs.ContainerOnStartMessage) sharedStructs.ContainerOnStartMessageResponse {
          logging.LogInfo("started", "inputMsg", message)
          return sharedStructs.ContainerOnStartMessageResponse{}
       },
       GetIDPMetadata: func(message authstructs.GetIDPMetadataMessage) authstructs.GetIDPMetadataMessageResponse {
          response := authstructs.GetIDPMetadataMessageResponse{
             Success: false,
          }
          err := initializeSAMLSP(authName, message.ServerName)
          if err != nil {
             response.Error = err.Error()
             return response
          }
          buf, err := xml.MarshalIndent(samlSP.ServiceProvider.Metadata(), "", " ")
          if err != nil {
             response.Error = err.Error()
             return response
          }
          response.Success = true
          response.Metadata = string(buf)
          return response
       },
...
}
```
{% endtab %}
{% endtabs %}

## Auth Functionality

There are two forms of auth containers - Identity Provider (IDP) SSO Services and Non-IDP checks. The IDP and NonIDP Services are available as options in the Logon UI page as long as the containers are online. If the auth container goes offline, then the option will disappear from the Logon page and you will have to fallback to the normal Mythic logon procedures.&#x20;

### IDP SSO Services

In these cases we are going to use the auth container to forward Mythic to a remote, SSO Identity Provider. This tends to have a few components:

* The SSO Identity Provider needs to get some sort of metadata from the auth container to register it as a trusted partner (i.e. `GetIDPMetadata`)
* The user needs to be redirected to this IDP to kick off the auth flow (username/password, MFA, duo prompts, etc) (i.e. `GetIDPRedirect`)
* After the IDP has determined if you're authorized or not, the result needs to be POSTed back to Mythic (and thus to the auth container) (i.e. `ProcessIDPResponse`). The end result here has to return a message back to Mythic to inform Mythic of two things:
  * was the authentication successful or not
  * what is the email associated with the user that just authenticated

The `IDPServices` array identifies which options are displayed to the user when they try to log in:

```
IDPServices:    []string{"ADFS"},
```

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

Selecting the `ADFS - MyAuthProvder` kicks off a request to the `GetIDPRedirect` function.

### Non-IDP SSO Services

The flow here is almost exactly the same as the IDP SSO Services, but instead of IDP, we have NonIDP everywhere. However, since we're not redirecting the user to an SSO service, we need to specify what information we need the user to provide. The `GetNonIDPRedirect` returns an array of fields that we want the user to specify:

```go
GetNonIDPRedirect: func(message authstructs.GetNonIDPRedirectMessage) authstructs.GetNonIDPRedirectMessageResponse {
    return authstructs.GetNonIDPRedirectMessageResponse{
       Success:       true,
       RequestFields: []string{"username", "password", "OTP"},
    }
},
```

This generates three fields in Mythic's logon page for `username`, `password`, and `OTP`. Once the user provides their data and click "login", then the containers `ProcessNonIDPResponse` function is called.

Just like in the IDP SSO Services function, at the end of this we need to return if auth was successful or not and the email of the user that was authenticated.
