---
layout: post
title: Secure your Logic Apps with Azure AD and API Management
tags:
    - Azure
    - Logic Apps
    - API Management
    - Azure AD
---

### JWT Validation

Integrating Azure Active Directory and other OpenID providers with Azure API Management is relativly easy with Azure API Management (APIM). Follow this [How To](https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-protect-backend-with-aad) to setup the required configuration. You can then validate a JSON Web Token (JWT) with [APIM access restriction policy](https://docs.microsoft.com/en-us/azure/api-management/api-management-access-restriction-policies#a-namevalidatejwta-validate-jwt). A simple example for Azure Active Directory will look like this:

```xml
<validate-jwt header-name="Authorization" require-scheme="Bearer" 
              failed-validation-httpcode="401"
              failed-validation-error-message="Unauthorized. Access token is missing or invalid.">
    <openid-config url="https://login.windows.net/tenant.onmicrosoft.com/.well-known/openid-configuration"/>
    <audiences>
        <audience>https://tenant.onmicrosoft.com/APIMAADDemo</audience>
    </audiences>
</validate-jwt> 
```

That's good, we have now have an API proxy that will control that the calling user is authentified and come from a known place. But if you try to add this policy to an API calling a Logic App, you will receive this cryptic error:

```json
{
  "error": {
    "code": "DirectApiAuthorizationRequired",
    "message": "The request must be authenticated only by Shared Access scheme."
  }
}
```

This is due to the fact that Logic Apps are not able to handle the ```Authorization``` HTTP header. We can easily go through by adding another policy to our inbound section:

```xml
<set-header name="Authorization" exists-action="delete"/>
```

### Pass JWT claims to a Logic App

Now, we can call our Logic Apps with success. But what if we need to pass information from the JWT Token to our workflow? For example, if we need to retrieve data based on the calling user.
To do that we will need to extract the data out of the JWT Token. For information, this is how an Azure AD token looks. For the sake of this blog, I will pass the ```upn``` and the ```name``` to the Logic App.

```json
{
    "aud": "https://tenant.onmicrosoft.com/APIMAADDemo",
    "iss": "https://sts.windows.net/.../",
    "iat": 1497517398,
    "nbf": 1497517398,
    "exp": 1497521298,
    "acr": "1",
    "aio": "..",
    "amr": [ "pwd" ],
    "appid": "...",
    "appidacr": "1",
    "family_name": "Bruttin",
    "given_name": "Sacha",
    "ipaddr": "0.0.0.0",
    "name": "Sacha Bruttin",
    "oid": "...",
    "platf": "3",
    "scp": "user_impersonation",
    "sub": "...",
    "tid": "...",
    "unique_name": "sbruttin@tenant.onmicrosoft.com",
    "upn": "sbruttin@tenant.onmicrosoft.com",
    "ver": "1.0"
}
```

<div class="notice--info">
If you want to know what your token contains you can copy/paste it to <a href="https://jwt.io">https://jwt.io</a>. The content of a token emitted by Azure AD is <a href="https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-token-and-claims">documented here</a>.
</div>

We will add another set of policies to our inbound section. First, we need to extract the value from the JWT Token. We can access the ```Authorization``` property on the Headers object that is part of the Request. We need to split the content on a ```[space]``` because the token will be preceded by the scheme (Bearer). The ```AsJwt``` method will convert the string into a JWT Token object we can read on the claim by it name. Then we will be able to add these values to the HTTP header using the ```set-header``` policy.

```xml
<!-- Extract the data into the context variables -->
<set-variable name="x-upn" value="@(context.Request.Headers["Authorization"].First().Split(' ')[1].AsJwt()?.Claims["upn"].FirstOrDefault())"/>
<set-variable name="x-username" value="@(context.Request.Headers["Authorization"].First().Split(' ')[1].AsJwt()?.Claims["name"].FirstOrDefault())"/>

<!-- Add new values to the HTTP Header -->
<set-header name="X-UPN" exists-action="override">
    <value>@((string)context.Variables["x-upn"])</value>
</set-header>
<set-header name="X-Username" exists-action="override">
    <value>@((string)context.Variables["x-username"])</value>
</set-header>
```

Now we need to read this values in our Logic Apps. It is in fact quite simple. The Request Trigger will pass the Headers to your next action, we will add a ```Parse JSON``` action right after it. We will use this schema to parse only our custom attributes:

```json
{
  "properties": {
    "X-UPN": {
      "type": "string"
    },
    "X-Username": {
      "type": "string"
    }
  },
  "type": "object"
}
```

Our properties are now available for the rest of the workflow.

![Great Logic Apps](/public/images/2017-06-apim-jwt-logicapps/logic-apps.png)

### What's next?

Our Logic Apps is now ready and we can test if from the developer portal. As you see, it easy to leverage APIM to secure our Logic Apps. The last step is to limit the access to our Logic Apps by restricting the calls to the IP addresses of the API Management portal. 

