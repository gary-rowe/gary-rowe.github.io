---
author: Gary Rowe
title: Dropwizard with OpenID
layout: post
tags:
  - HowTo
  - Dropwizard
  - OpenID
  - Tutorial
redirect_from: /agilestack/2012/12/12/dropwizard-with-openid/
---
[Dropwizard](http://dropwizard.codahale.com) is an excellent framework to create lightweight web services with. Out of the box it comes with a wide range of useful features that just make developing web services a pleasure. In particular, support for Basic authentication and OAuth2 are pretty much required for any web service that is public facing these days - and why aren't you employing secure web services internally? Criticism aside, one common authentication technique appears to be missing: [OpenID](https://openid.net). 

[Just gimme the code!](https://github.com/gary-rowe/DropwizardOpenID)

## What is the difference between OpenId and OAuth?

Many people get confused about the difference between OpenID and OAuth so I'll quickly summarise it here. OpenID is about authentication (ie. proving who you are) whereas OAuth is about authorisation (ie. to grant access to something without having to deal with the original authentication). 

In both cases, a third party (such as Google or Yahoo) is involved which performs the actual authentication and then shares a token with the site you're visiting to avoid repeat authentications. OpenID also allows you to share some of your personal information with this site.

## A simple explanation of how OpenID works

Let's walk through a typical OpenID authentication between you and a merchant site with Google acting as the OpenID provider. 

You want to visit your private account page. The merchant has no idea who you are and so bounces you back to an authentication screen. You see a list of suggested OpenID authenticators, probably in the form of clickable images. You click the Google image which has the effect of POSTing a form to the merchant with a field defined as `identifier=https://www.google.com/accounts/o8/id`. 

While you wait for a response the merchant requests against this identifier (an OpenID discovery operation). Their response from Google contains a bunch of named identifiers that can be used to perform the actual authentication. This indirection allows Google to change their OpenID support endpoints without the merchant having to update their site.

The merchant now uses this information to build a browser redirect response to you. You bounce off to Google, perform your authentication (potentially over many requests) and eventually Google authenticates you and issues a redirect response which bounces you back to the merchant. This is all co-ordinated through the OpenID protocol which mandates a particular URI structure. This protocol also allows the merchant to add in some extra requests for information - such as asking for your email address, full name and so on. 

From a resource perspective it is obvious that there are two endpoints that a merchant site has to provide:

1. the authentication endpoint to receive the OpenID identifier ("I'll be authenticating with Google today...")
1. the verification endpoint to receive the redirect from Google containing the result of the authentication and matching this up with their own records 

## Integrating with Dropwizard

The most important code is the excellent [OpenID4Java library](http://code.google.com/p/openid4java/) which does all the heavy lifting so that you don't have to. To bring it in add a Maven dependency entry like this:

```xml
<!-- OpenID heavy lifting -->
<dependency>
  <groupId>org.openid4java</groupId>
  <artifactId>openid4java-consumer</artifactId>
  <version>0.9.6</version>
  <type>pom</type>
</dependency>
```

Now you can write an authentication endpoint that contains the following code:

```java
/**
 * Handles the authentication request from the user after they select their OpenId server
 *
 * @param identifier The identifier for the OpenId server
 *
 * @return A redirection or a form view containing user-specific permissions
 */
@POST
public Response authenticationRequest(
  @Context
  HttpServletRequest request,
  @FormParam("identifier")
  String identifier
) {

  UUID sessionToken = UUID.randomUUID();

  try {

    // The OpenId server will use this endpoint to provide authentication
    // Parts of this may be shown to the user
    final String returnToUrl;
    if (request.getServerPort() == 80) {
      returnToUrl = String.format(
        "http://%s/openid/verify?token=%s",
        request.getServerName(),
        sessionToken);
    } else {
      returnToUrl = String.format(
        "http://%s:%d/openid/verify?token=%s",
        request.getServerName(),
        request.getServerPort(),
        sessionToken);
    }

    log.debug("Return to URL '{}'", returnToUrl);

    // Create a consumer manager for this specific request and cache it
    // (this is to preserve session state such as nonce values etc)
    ConsumerManager consumerManager = new ConsumerManager();
    openIDCache.putConsumerManager(sessionToken, consumerManager);

    // Perform discovery on the user-supplied identifier
    List discoveries = consumerManager.discover(identifier);

    // Attempt to associate with the OpenID provider
    // and retrieve one service endpoint for authentication
    DiscoveryInformation discovered = consumerManager.associate(discoveries);

    // Create a memento to rebuild the discovered information in a subsequent request
    DiscoveryInformationMemento memento = new DiscoveryInformationMemento();
    if (discovered.getClaimedIdentifier() != null) {
      memento.setClaimedIdentifier(discovered.getClaimedIdentifier().getIdentifier());
    }
    memento.setDelegate(discovered.getDelegateIdentifier());
    if (discovered.getOPEndpoint() != null) {
      memento.setOpEndpoint(discovered.getOPEndpoint().toString());
    }
    memento.setTypes(discovered.getTypes());
    memento.setVersion(discovered.getVersion());

    // Create a temporary User to preserve state between requests without
    // using a session (we could be in a cluster)
    User tempUser = new User(null, sessionToken);
    tempUser.setOpenIDDiscoveryInformationMemento(memento);
    tempUser.setSessionToken(sessionToken);
    userRepository.save(tempUser);
    //userService.create(tempUser);

    // Build the AuthRequest message to be sent to the OpenID provider
    AuthRequest authReq = consumerManager.authenticate(discovered, returnToUrl);

    // Build the FetchRequest containing the information to be copied
    // from the OpenID provider
    FetchRequest fetch = FetchRequest.createFetchRequest();
    // Attempt to decode each entry
    if (identifier.startsWith(GOOGLE_ENDPOINT)) {
      fetch.addAttribute("email", "http://axschema.org/contact/email", true);
      fetch.addAttribute("firstName", "http://axschema.org/namePerson/first", true);
      fetch.addAttribute("lastName", "http://axschema.org/namePerson/last", true);
    } else if (identifier.startsWith(YAHOO_ENDPOINT)) {
      fetch.addAttribute("email", "http://axschema.org/contact/email", true);
      fetch.addAttribute("fullname", "http://axschema.org/namePerson", true);
    } else { // works for myOpenID
      fetch.addAttribute("fullname", "http://schema.openid.net/namePerson", true);
      fetch.addAttribute("email", "http://schema.openid.net/contact/email", true);
    }

    // Attach the extension to the authentication request
    authReq.addExtension(fetch);

    // Redirect the user to their OpenId server authentication process
    return Response
      .seeOther(URI.create(authReq.getDestinationUrl(true)))
      .build();

  } catch (MessageException e1) {
    log.error("MessageException:", e1);
  } catch (DiscoveryException e1) {
    log.error("DiscoveryException:", e1);
  } catch (ConsumerException e1) {
    log.error("ConsumerException:", e1);
  }
  return Response.ok().build();
}
```

Then add in a verification endpoint that looks like this:

```java
/**
 * Handles the OpenId server response to the earlier AuthRequest
 *
 * @return The OpenId identifier for this user if verification was successful
 */
@GET
@Path("/verify")
public Response verifyOpenIdServerResponse(
  @Context HttpServletRequest request,
  @QueryParam("token") String rawToken) {

  // Retrieve the previously stored discovery information from the temporary User
  if (rawToken == null) {
    log.debug("Authentication failed due to no session token");
    throw new WebApplicationException(Response.Status.UNAUTHORIZED);
  }

  // Build a session token from the request
  UUID sessionToken = UUID.fromString(rawToken);

  // Attempt to locate the consumer manager by the session token
  Optional<ConsumerManager> consumerManagerOptional = openIDCache.getConsumerManager(sessionToken);
  if (!consumerManagerOptional.isPresent()) {
    log.debug("Authentication failed due to no consumer manager matching session token {}", rawToken);
    throw new WebApplicationException(Response.Status.UNAUTHORIZED);
  }
  ConsumerManager consumerManager = consumerManagerOptional.get();

  // Attempt to locate the user by the session token
  Optional<User> tempUserOptional = userReadService.getBySessionToken(sessionToken);
  if (!tempUserOptional.isPresent()) {
    log.debug("Authentication failed due to no temp User matching session token {}", rawToken);
    throw new WebApplicationException(Response.Status.UNAUTHORIZED);
  }
  User tempUser = tempUserOptional.get();

  // Retrieve the discovery information
  final DiscoveryInformationMemento memento = tempUser.getOpenIDDiscoveryInformationMemento();
  Identifier identifier = new Identifier() {
    @Override
    public String getIdentifier() {
      return memento.getClaimedIdentifier();
    }
  };

  DiscoveryInformation discovered;
  try {
    discovered = new DiscoveryInformation(
      URI.create(memento.getOpEndpoint()).toURL(),
      identifier,
      memento.getDelegate(),
      memento.getVersion(),
      memento.getTypes()
    );
  } catch (DiscoveryException e) {
    throw new WebApplicationException(e, Response.Status.UNAUTHORIZED);
  } catch (MalformedURLException e) {
    throw new WebApplicationException(e, Response.Status.UNAUTHORIZED);
  }

  // Extract the receiving URL from the HTTP request
  StringBuffer receivingURL = request.getRequestURL();
  String queryString = request.getQueryString();
  if (queryString != null && queryString.length() > 0) {
    receivingURL.append("?").append(request.getQueryString());
  }
  log.debug("Receiving URL = '{}", receivingURL.toString());

  // Extract the parameters from the authentication response
  // (which comes in as a HTTP request from the OpenID provider)
  ParameterList parameterList = new ParameterList(request.getParameterMap());

  try {

    // Verify the response
    // ConsumerManager needs to be the same (static) instance used
    // to place the authentication request
    // This could be tricky if this service is load-balanced
    VerificationResult verification = consumerManager.verify(
      receivingURL.toString(),
      parameterList,
      discovered);

    // Examine the verification result and extract the verified identifier
    Optional<Identifier> verified = Optional.fromNullable(verification.getVerifiedId());
    if (verified.isPresent()) {
      // Verified
      AuthSuccess authSuccess = (AuthSuccess) verification.getAuthResponse();

      // We have successfully authenticated so remove the temp user
      // and replace it with a potentially new one
      userRepository.hardDelete(tempUser);

      tempUser = new User(null, UUID.randomUUID());
      tempUser.setOpenIDIdentifier(verified.get().getIdentifier());

      // Provide a basic authority in light of successful authentication
      tempUser.getAuthorities().add(Authority.ROLE_PUBLIC);

      // Extract additional information
      if (authSuccess.hasExtension(AxMessage.OPENID_NS_AX)) {
        tempUser.setEmailAddress(extractEmailAddress(authSuccess));
        tempUser.setFirstName(extractFirstName(authSuccess));
        tempUser.setLastName(extractLastName(authSuccess));
      }
      log.info("Extracted a temporary {}", tempUser);

      // Search for a pre-existing User matching the temp User
      Optional<User> userOptional = userReadService.getByOpenIDIdentifier(tempUser.getOpenIDIdentifier());
      User user;
      if (!userOptional.isPresent()) {
        // This is either a new registration or the OpenID identifier has changed
        if (tempUser.getEmailAddress() != null) {
          userOptional = userReadService.getByEmailAddress(tempUser.getEmailAddress());
          if (!userOptional.isPresent()) {
            // This is a new User
            log.debug("Registering new {}", tempUser);
            user = tempUser;
          } else {
            // The OpenID identifier has changed so update it
            log.debug("Updating OpenID identifier for {}", tempUser);
            user = userOptional.get();
            user.setOpenIDIdentifier(tempUser.getOpenIDIdentifier());
          }

        } else {
          // No email address to use as backup
          log.warn("Rejecting valid authentication. No email address for {}");
          throw new WebApplicationException(Response.Status.UNAUTHORIZED);
        }
      } else {
        // The User has been located by their OpenID identifier
        log.debug("Found an existing User using OpenID identifier {}", tempUser);
        user = userOptional.get();

      }

      // Persist the user with the current session token
      user.setSessionToken(sessionToken);
      userRepository.save(user);

      // Create a suitable view for the response
      // The session token has changed so we create the base model directly
      BaseModel model = new BaseModel();
      model.setUser(user);

      // Authenticated
      View view = new PrivateFreemarkerView<BaseModel>("private/home.ftl", model);

      // Refresh the session token cookie
      return Response
        .ok()
        .cookie(replaceSessionTokenCookie(Optional.of(user)))
        .entity(view)
        .build();

    } else {
      log.debug("Failed verification");
    }
  } catch (OpenIDException e) {
    // present error to the user
    log.error("OpenIDException", e);
  }

  // Must have failed to be here
  throw new WebApplicationException(Response.Status.UNAUTHORIZED);
}
```

Which gives you working OpenID in your project in a jiffy. 

Of course, I've omitted huge tracts of supporting code to keep this article brief, but the [DropwizardOpenID project on GitHub](https://github.com/gary-rowe/DropwizardOpenID) has all of that code ready to go out of the box. You could use it as your initial basis for creating a Dropwizard project with OpenID support, and then expand upon it as you introduce your own work.

## Final words

Dropwizard comes with a lot of built in support for authentication, but OpenID is still very useful to developers and anything that can make the implementation easier has got to be useful. 

If you like the sample code, please star it in GitHub - it would be nice to know that it's popular. Also, if there any improvements that you'd like to see made do feel free to add an Issue and I'll take a look at it.
