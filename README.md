## Spring Auth2 Demo Project
[[_social_login_first_party]]
= Login with GitHub

In this section, you'll modify the <<_social_login_logout,logout>> app you built already, adding a sticker page so that the end-user can choose between multiple sets of credentials.

Let's add Google as a second option for the end user.

[[google-initial-setup]]
== Initial setup

To use Google's OAuth 2.0 authentication system for login, you must set up a project in the Google API Console to obtain OAuth 2.0 credentials.

NOTE: https://developers.google.com/identity/protocols/OpenIDConnect[Google's OAuth 2.0 implementation] for authentication conforms to the
 https://openid.net/connect/[OpenID Connect 1.0] specification and is https://openid.net/certification/[OpenID Certified].

Follow the instructions on the https://developers.google.com/identity/protocols/OpenIDConnect[OpenID Connect] page, starting in the section, "Setting up OAuth 2.0".

After completing the "Obtain OAuth 2.0 credentials" instructions, you should have a new OAuth Client with credentials consisting of a Client ID and a Client Secret.

[[google-redirect-uri]]
== Setting the redirect URI

Also, you'll need to supply a redirect URI, as you did for GitHub earlier.

In the "Set a redirect URI" sub-section, ensure that the *Authorized redirect URIs* field is set to `http://localhost:8080/login/oauth2/code/google`.

== Adding the Client Registration

Then, you need to configure the client to point Google.
Because Spring Security is built with multiple clients in mind, you can add our Google credentials alongside the ones you created for GitHub:

.application.yml
[source,yaml]
----
spring:
  security:
    oauth2:
      client:
        registration:
          github:
            clientId: github-client-id
            clientSecret: github-client-secret
          google:
            client-id: google-client-id
            client-secret: google-client-secret
----

As you can see, Google is another provider that Spring Security ships out-of-the-box support for.

== Adding the Login Link

In the client, the change is trivial - you can just add another link:

.index.html
[source,html]
----
<div class="container unauthenticated">
  <div>
    With GitHub: <a href="/oauth2/authorization/github">click here</a>
  </div>
  <div>
    With Google: <a href="/oauth2/authorization/google">click here</a>
  </div>
</div>
----

NOTE: The final path in the URL should match the client registration id in `application.yml`.

TIP: Spring Security ships with a default provider selection page that can be reached by pointing to `/login` instead of `/oauth2/authorization/{registrationId}`.

== How to Add a Local User Database

Many applications need to hold data about their users locally, even if authentication is delegated to an external provider.
We don't show the code here, but it is easy to do in two steps.

1. Choose a backend for your database, and set up some repositories (using Spring Data, say) for a custom `User` object that suits your needs and can be populated, fully or partially, from external authentication.

2. Implement and expose `OAuth2UserService` to call the Authorization Server as well as your database.
  Your implementation can delegate to the default implementation, which will do the heavy lifting of calling the Authorization Server.
  Your implementation should return something that extends your custom `User` object and implements `OAuth2User`.

Hint: add a field in the `User` object to link to a unique identifier in the external provider (not the user's name, but something that's unique to the account in the external provider).
