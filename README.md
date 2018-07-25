# Exploring Azure AD
This is an exploration of Azure AD, notes taken and organized during reading [**Azure Active Directory for Developers Guide**](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-developers-guide). Since it is a huge guide, this summary could be helpful. Most important concepts are highlighted and re-organized from developer's approach.

## [Terminology](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-dev-glossary)
- **tenant:** An instance of an Azure AD directory is referred to as an Azure AD tenant. It provides several features, including:
    - a registry service for integrated applications
    - authentication of user accounts and registered applications
    -REST endpoints required to support various protocols including OAuth2 and SAML, including the authorization endpoint, token endpoint and the "common" endpoint used by multi-tenant applications.
    
- **authentication:** The act of challenging a party for legitimate credentials, providing the basis for creation of a security principal to be used for identity and access control. 

- **authorization:** The act of granting an authenticated security principal permission to do something. 

- **sign-in:** The process of a client application initiating end-user authentication and capturing related state, for the purpose of acquiring a security token and scoping the application session to that state. State can include artifacts such as user profile information, and information derived from token claims.

  The sign-in function of an application is typically used to implement single-sign-on (SSO).

- **sign-out:** The process of unauthenticating an end user, detaching the user state associated with the client application session during sign-in.

- **single tenant application:** A single tenant application is intended for use in one organization. It only needs to be accessed by users in one directory, and as a result, to be provisioned in one directory. These applications are typically registered by a developer in the organization.

- **multi-tenant application:** A class of application that enables sign in and consent by users provisioned in any Azure AD tenant, including tenants other than the one where the client is registered. 

    A multi-tenant application is intended for use in many organizations, not just one organization. Multi-tenant applications need to be provisioned in each directory where they will be used.
    
    When a user or administrator from a different organization signs up to use the application, they are presented with a dialog that displays the permissions the application requires. The user or administrator can then consent to the application, which gives the application access to the stated data, and finally registers the application in their directory. 
  
- **client application:** As defined by the OAuth2 Authorization Framework, an application that makes protected resource requests on behalf of the resource owner. 

  A client application requests authorization from a resource owner to participate in an OAuth2 authorization grant flow, and may access APIs/data on the resource owner's behalf. 
  
- **resource owner:** As defined by the OAuth2 Authorization Framework, an entity capable of granting access to a protected resource. When the resource owner is a person, it is referred to as an end user.

- **authorization server:** As defined by the OAuth2 Authorization Framework, the server responsible for issuing access tokens to the client after successfully authenticating the resource owner and obtaining its authorization. A client application interacts with the authorization server at runtime via its authorization and token endpoints, in accordance with the OAuth2 defined authorization grants.

  In the case of Azure AD application integration, Azure AD implements the authorization server role for Azure AD applications and Microsoft service APIs
  
- **resource server:** As defined by the OAuth2 Authorization Framework, a server that hosts protected resources, capable of accepting and responding to protected resource requests by client applications that present an access token. 
  A resource server exposes APIs and enforces access to its protected resources through scopes and roles, using the OAuth 2.0 Authorization Framework. 
  
- **authorization grant:** A credential representing the resource owner's authorization to access its protected resources, granted to a client application. The credential returned to the client is either an access token, or an authorization code (exchanged later for an access token), depending on the type of authorization grant used.
  
- **authorization endpoint:** One of the endpoints implemented by the authorization server, used to interact with the resource owner in order to provide an authorization grant during an OAuth2 authorization grant flow. 

- **token endpoint:** One of the endpoints implemented by the authorization server to support OAuth2 authorization grants. Depending on the grant, it can be used to acquire an access token (and related "refresh" token) to a client, or ID token when used with the OpenID Connect protocol.

- **authorization code:** A short lived "token" provided to a client application by the authorization endpoint, as part of the "authorization code" flow. The code is returned to the client application in response to authentication of a resource owner, indicating the resource owner has delegated authorization to access the requested resources. As part of the flow, the code is later redeemed for an access token.

- **access token:** A type of security token issued by an authorization server, and used by a client application in order to access a protected resource server. Typically in the form of a JSON Web Token (JWT), the token embodies the authorization granted to the client by the resource owner, for a requested level of access. 
  "Authorization code" authorization grant, the end user authenticates first as the resource owner, delegating authorization to the client to access the resource. The client authenticates afterward when obtaining the access token. 
  "Client credentials" authorization grant, the client provides the sole authentication, functioning without the resource-owner's authentication/authorization.
  
- **ID token:** An OpenID Connect security token provided by an authorization server's authorization endpoint, which contains claims pertaining to the authentication of an end user resource owner. Like an access token, ID tokens are also represented as a digitally signed JSON Web Token (JWT). Unlike an access token though, an ID token's claims are not used for purposes related to resource access and specifically access control.

- **security token:** A signed document containing claims, such as an OAuth2 token or SAML 2.0 assertion. For an OAuth2 authorization grant, an access token (OAuth2) and an ID Token are types of security tokens, both of which are implemented as a JSON Web Token (JWT).

- **application registration:** In order to allow an application to integrate with and delegate Identity and Access Management functions to Azure AD, it must be registered with an Azure AD tenant. 
  When you register/update an application in the Azure portal, the portal creates/updates both an application object and a corresponding service principal object for that tenant. Similar to the way a service principal object is used to represent an application instance, a user principal object is another type of security principal, which represents a user. 

- **application id (client id):** The unique identifier Azure AD issues to an application registration. 

- **claim:** A security token contains claims, which provide assertions about one entity (such as a client application or resource owner) to another entity (such as the resource server). Claims are name/value pairs that relay facts about the token subject.

- **consent:** The process of a resource owner granting authorization to a client application, to access protected resources under specific permissions, on behalf of the resource owner.
  Depending on the permissions requested by the client, an administrator or user will be asked for consent to allow access to their organization/individual data respectively.
  
- **permissions:** A client application gains access to a resource server by declaring permission requests. Two types are available:
    - "Delegated" permissions, which specify scope-based access using delegated authorization from the signed-in resource owner, are presented to the resource at run-time as "scp" claims in the client's access token.
    - "Application" permissions, which specify role-based access using the client application's credentials/identity, are presented to the resource at run-time as "roles" claims in the client's access token.
  They also surface during the consent process, giving the administrator or resource owner the opportunity to grant/deny the client access to resources in their tenant.
  
- **roles:** Like scopes, roles provide a way for a resource server to govern access to its protected resources. Roles are resource-defined strings.

- **scopes:** Like roles, scopes provide a way for a resource server to govern access to its protected resources. Scopes are used to implement scope-based access control, for a client application that has been given delegated access to the resource by its owner. Scopes are resource-defined strings 

## Overview
Azure AD is the identity provider, responsible for verifying the identity of users and applications that exist in an organization’s directory, and ultimately issuing security tokens upon successful authentication of those users and applications.

An application that wants to outsource authentication to Azure AD must be registered in Azure AD, which registers and uniquely identifies the app in the directory.

Developers can use the open-source Azure AD authentication libraries (ADAL) to make authentication easy by handling the protocol details for you.

Once a user has been authenticated, the application must validate the user’s security token to ensure that authentication was successful. 

The flow of requests and responses for the authentication process is determined by the authentication protocol that was used, such as OAuth 2.0, OpenID Connect, WS-Federation, or SAML 2.0. 

Azure AD supports the OAuth 2.0 and OpenID Connect standards that make extensive use of bearer tokens, including bearer tokens represented as JWTs. Always ensure that your application transmits and stores bearer tokens in a secure manner (such as transport layer security (HTTPS)). 

## Registering an Application in Azure AD
Any application that outsources authentication to Azure AD must be registered in a directory. This step involves telling Azure AD about your application, including the URL where it’s located, the URL to send replies after authentication, the URI to identify your application, and more. Azure AD needs to communicate with the application when handling sign-on or exchanging tokens. 

- **Application ID URI:** The identifier for an application. This value is sent to Azure AD during authentication to indicate which application the caller wants a token for. 

- **Reply URL and Redirect URI:** For a web API or web application, the Reply URL is the location where Azure AD will send the authentication response, including a token if authentication was successful. 

- **Application ID:** The ID for an application, which is generated by Azure AD when the application is registered. When requesting an authorization code or token, the Application ID and Key are sent to Azure AD during authentication.

- **Key:** The key that is sent along with an Application ID when authenticating to Azure AD to call a web API.

**Single Tenant:** If you are building an application just for your organization, it must be registered in your company’s directory by using the Azure portal.

**Multi-Tenant:** If you are building an application that can be used by users outside your organization, it must be registered in your company’s directory, but also must be registered in each organization’s directory that will be using the application. To make your application available in their directory, you can include a sign-up process for your customers that enables them to consent to your application. When they sign up for your application, they will be presented with a dialog that shows the permissions the application requires, and then the option to consent. Depending on the required permissions, an administrator in the other organization may be required to give consent. When the user or administrator consents, the application is registered in their directory. 

## Common Endpoint vs Tenant Endpoint

Some additional considerations arise when developing a multi-tenant application instead of a single tenant application. 

If you are making your application available to users in multiple directories, you need a mechanism to determine which tenant they’re in.

A single tenant application only needs to look in its own directory for a user, while a multi-tenant application needs to identify a specific user from all the directories in Azure AD. 

To accomplish this task, Azure AD provides a common authentication endpoint where any multi-tenant application can direct sign-in requests, instead of a tenant-specific endpoint. 

Common endpoint is **https://login.microsoftonline.com/common** for all directories in Azure AD, whereas a tenant-specific endpoint might be **https://login.microsoftonline.com/contoso.onmicrosoft.com**.

**The common endpoint is especially important to consider when developing your application because you’ll need the necessary logic to handle multiple tenants during sign-in, sign-out, and token validation.**

## Usage Scenario: a web application that needs to get resources from a web API

![diagram](https://docs.microsoft.com/en-us/azure/active-directory/develop/media/active-directory-authentication-scenarios/web_app_to_web_api.png)

There are two identity types that the web application can use to authenticate and call the web API:

- **Application identity:** This scenario uses OAuth 2.0 client credentials grant to authenticate as the application and access the web API. 

- **Delegated user identity:** This scenario can be accomplished in two ways: OpenID Connect, and OAuth 2.0 authorization code grant with a confidential client. The web application obtains an access token for the user, which proves to the web API that the user successfully authenticated to the web application and that the web application was able to obtain a delegated user identity to call the web API. This access token is sent in the request to the web API, which authorizes the user and returns the desired resource.

### Application Identity with OAuth 2.0 Client Credentials grant
A user is signed in to Azure AD in the web application.

The web application needs to acquire an access token so that it can authenticate to the web API and retrieve the desired resource. It makes a request to Azure AD’s token endpoint, providing the credential, Application ID, and web API’s application ID URI.

Azure AD authenticates the application and returns a JWT access token that is used to call the web API.

Over HTTPS, the web application uses the returned JWT access token to add the JWT string with a “Bearer” designation in the Authorization header of the request to the web API. The web API then validates the JWT token, and if validation is successful, returns the desired resource.

### Delegated User Identity with OpenID Connect
A user is signed in to a web application using Azure AD. If the user of the web application has not yet consented to allowing the web application to call the web API on its behalf, the user will need to consent. The application will display the permissions it requires, and if any of these are administrator-level permissions, a normal user in the directory will not be able to consent. This consent process only applies to multi-tenant applications, not single tenant applications, as the application will already have the necessary permissions. When the user signed in, the web application received an ID token with information about the user, as well as an authorization code.

Using the authorization code issued by Azure AD, the web application sends a request to Azure AD’s token endpoint that includes the authorization code, details about the client application (Application ID and redirect URI), and the desired resource (application ID URI for the web API).

The authorization code and information about the web application and web API are validated by Azure AD. Upon successful validation, Azure AD returns two tokens: a JWT access token and a JWT refresh token.

Over HTTPS, the web application uses the returned JWT access token to add the JWT string with a “Bearer” designation in the Authorization header of the request to the web API. The web API then validates the JWT token, and if validation is successful, returns the desired resource.

**Token expiration:** When the web application uses its authorization code to get a JWT access token, it also receives a JWT refresh token. When the access token expires, the refresh token can be used to re-authenticate the user without requiring them to sign in again. This refresh token is then used to authenticate the user, which results in a new access token and refresh token.

## Azure AD v1.0 Endpoint vs v2.0 Endpoint
Azure AD v1.0 endpoint supports only Microsoft work or school accounts. v2.0 endpoint also supports personal Microsoft accounts. v2.0 endpoint currently have some limitations.

## Hands-On / A PassportJS strategy for Microsoft Azure AD: [**passport-azure-ad**](https://github.com/AzureAD/passport-azure-ad)
[Example Code](https://github.com/AzureADQuickStarts/WebApp-OpenIDConnect-NodeJS)
 
- [Register the app.](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-devquickstarts-openidconnect-nodejs#step-1-register-an-app) 

    *Client id* and *client secret* should be obtained at this step.
 
- **passport-azure-ad** should be added/installed as dependency.
 
- Use **OIDCStrategy** after requring it:

	``` js
	var OIDCStrategy = require('passport-azure-ad').OIDCStrategy;
	```

- Strategies in passport require a *validate* function that accepts credentials (such as an OpenID identifier), and invokes a callback with a user object. [verify callback](https://github.com/AzureAD/passport-azure-ad#5113-verify-callback)
 
    Looking at the strategy, you see that we pass it a function that has a token and a done as the parameters. The strategy comes back to us after it does its work. Then we want to store the user and stash the token so we don't need to ask for it again.
	
- Add the methods that enable to track the signed-in users as required by Passport. These methods include serializing and deserializing the user's information.

	To support persistent sign-in sessions, Passport needs to be able to serialize users into the session and deserialize them out of the session. Typically, this is done simply by storing the user ID when serializing and finding the user by ID when deserializing.
 
- Configure Express. Initialize Passport. Also use *passport.session()* middleware, to support persistent login sessions (recommended).

- Add the routes that hand off the actual sign-in requests to the **passport-azure-ad** engine.

- Use Passport to issue sign-in and sign-out requests to Azure AD.

## Troubleshooting

### Proxy Issue with http(s) request
I had proxy issue when *code* is used in *responseType* behind corporate proxy. The reason is passport.authenticate() method which is used in callback uses *http(s).request* and somehow it could not consider my proxy environment variables. 

In theory, npm uses proxies defined in .npmrc file which might be set by using npm config set.. command. Nodejs uses environment variable settings.

I used **global-tunnel-ng** to ensure nodejs is using my proxies. Then, since **global-tunnel-ng** does not currently support *no-proxy* environment variable, my internal config function at *localhost* is not reachable behind proxy. To fix that, i initialized **global-tunnel-ng** just before passport.authenticate() in callback method and ended it right afterwards.

## Refresh Token Mechanism
[Passport doesn't get involved in this process, because it is separate from authentication.](https://stackoverflow.com/a/15627084)

[passport-oauth2-refresh](https://github.com/fiznool/passport-oauth2-refresh) is promising, however it does not accept *OIDCStrategy*. However, i still could use it by encapsulating oauth2 related code and only passing required parameters. 

- [ ] I am planning to publish a PR for above modification to [passport-oauth2-refresh](https://github.com/fiznool/passport-oauth2-refresh). 

## Code Samples
[**Azure AD Node.js web app getting started:**](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-devquickstarts-openidconnect-nodejs) Use Passport to:
- Sign the user in to the app with Azure Active Directory (Azure AD).
- Display information about the user.
- Sign the user out of the app.


[My PassportJs POC](https://github.com/hlltarakci/poc_passportjs)
