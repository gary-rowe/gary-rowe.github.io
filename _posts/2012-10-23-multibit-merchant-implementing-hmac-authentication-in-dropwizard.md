---
author: Gary Rowe
title: MultiBit Merchant - Implementing HMAC authentication in Dropwizard
layout: post
tags:
  - HowTo
  - Dropwizard
  - HMAC
  - MBM
redirect_from: /agilestack/2012/10/23/multibit-merchant-implementing-hmac-authentication-in-dropwizard/
---

As part of my ongoing open source project [MultiBit Merchant (MBM)][2] I am keeping a journal of my discoveries and thoughts along the way. This one deals with how I implemented HMAC authentication for Dropwizard as part of the security for the MBM RESTful API. There is a lot of ground to cover so this is going to be a long one. I’ve added lots of code examples but these will drift out of date so I’d recommend reviewing the source code of the MBM project to get the latest implementations.

### I’m in a hurry where can I just get everything right now?

The fastest way to get this into your own project would be to clone the MBM repo and then pick out the following classes and integrate them into your own package structure:

Production code:

* HmacServerAuthenticator
* HmacServerCredentials
* HmacServerRestrictedToInjectable
* HmacServerRestrictedToProvider
* HmacClientFilter
* HmacUtils
* MultiBitMerchantService – to show startup configuration

Test support code

* BaseResourceTest
* BaseJerseyHmacResourceTest
* AdminUserResourceTest – to demonstrate accessing a restricted resource during test

### I want the details – what’s the problem?

The MBM project consists of a client application that handles the pretty front end, and the MBM server application that only offers a RESTful API and does all the heavy lifting. This means that the client app has to pull double duty in order to allow site visitors to access resources on MBM through their own credentials, and to allow the client app itself to make authenticated requests to retrieve specialised information that site visitors don’t see.

### Why not Basic?

Basic authentication relies on a username and password being transmitted in each request with the only protection being Base64 encoding in the HTTP header section. It is trivial to eavesdrop on HTTP and as such Basic authentication should only be used on a secure connection. Even then, it is possible for a [man-in-the-middle attack to take place][3] with the evil Mallory rewriting the messages between the unsuspecting Alice and Bob.

### Why not OAuth2?

I found it far too complex to work with, and it seems that the [lead author and editor has serious reservations][4] about it too. My biggest problem was abandoning a very simple and scalable authentication mechanism like HMAC in favour of a complex network traffic generating alternative like OAuth2. Debugging a distributed application is hard enough without bouncing around a bunch of servers just to get an authentication to complete. Maybe when all the kinks have been ironed out and there are drop-in modules to support it then I’ll revisit it, but until then I’m going with HMAC.

### What is HMAC?

HMAC is a Hash-based Message Authentication Code. Breaking the acronym down it’s pretty obvious that a hash of some message is generated and the resulting code can be used for verifying the authenticity of the message. This is really handy for RESTful APIs because we want to be able to send messages with verifiable authenticity over HTTP for speed. If someone does intercept the message, they won’t be able to change the content, or replay the same request, unless they know a secret. And that secret is known only to the client and the server and never present in the message so it is inherently more resilient to attack.

### What is the general process for creating one?

Here is a quick walk-through of the process of a client making a request against the server

1. Build the client request
2. Create a simplified representation of the request by stringing together important parts of the request – this is the *canonical representation*
3. Append the secret key to the canonical representation
4. Perform a SHA-1 hash over the canonical representation
5. Add a new HTTP Authorization header containing this hash and an identifier – usually an API key

At the server end the objective is to rebuild the canonical representation exactly as the client would have built it. The general process goes like this:

1. Check the time information in the request is not too far in the past
2. Check any nonce (number used only once) values in the request have not been repeated (optional)
3. Use the API key in the request to locate the secret key in the server database
4. Re-create the canonical representation by stringing together various parts of the request from the client
5. Append the secret key to the canonical representation
6. Perform a SHA-1 hash over the canonical representation
7. Check that the hash is the same as the one reported in the header provided by the client

If the hashes match, then the request must have originated from someone who has access to the secret key, which should only be the client. You can consider the request authentic.

### Some security details

The reason for including and checking the HTTP Date header is to ensure that replay attacks are given a short window of opportunity. If it were otherwise then an old request could theoretically be recorded and replayed later and still have a chance of being successful, with potentially disastrous results. To avoid HTTP header re-writing in transit by innocent proxies or hosting environments, the date and time information should be included in its own header (e.g. X-HMAC-Date) and should be expressed in UTC.

In general, HTTP headers other than those mentioned, should not be included in the canonical representation. 

The HTTP method (GET, POST, PUT etc) should also be included to avoid manipulation that could send a replayed request to the wrong endpoint and cause problems.

It is always good to include a nonce value such as an ever-increasing number (like a millisecond timestamp) since this will be easy to compare with a previous known value and makes it harder for a replay attack to occur. This should be placed in its own header (e.g. X-HMAC-Nonce).

In POST and PUT requests the entire entity body should be included in the canonical representation so that any changes will invalidate the hash.

The full original request URI should also be included to ensure that the endpoint has not been modified in transit. Choosing the URL-decoded representation is often easier to work with.

There is no support for query-based authentication (where a GET request has the hash result encoded as query parameters) because this turns a URI into an authenticator rather than a resource identifier. It would have the side-effect of authenticated URIs being shared as permalinks which would later fail through a time-to-live threshold being exceeded.

### Where can I just copy-paste all that?

[HmacUtils.java is an implementation of the above][5] for the Jersey implementation of JAX-RS. This allows it to integrate smoothly with Dropwizard because it can be used as part of the client-side resource tests so you can test the authentication aspect of your resources.

### How do I integrate this with my own Dropwizard project?

You’ll need some more code to finish the job. The Jersey implementation of JAX-RS strongly favours the use of object injection using annotations. This is all managed through the use of `Injectable` and `InjectableProvider` which combine to provide a framework that supports an annotation.

Typically, security annotations (like those found in Spring Security) provide a list of authorities that have to be matched in order to authorise the user and allow access to the method. Often these authorities are listed as strings so that arbitrary authorities can be added to a database and supported as necessary. Obviously, using strings for this purpose can lead to complications with case sensitivity and spelling during development.

One way around this is to define an enum that contains all the necessary authorities. This provides a statically bound list so an IDE can offer up choices during development, and helps to create a concise representation of all the required authorities to support the system in a single place. An example is shown below.

```java
public enum Authority {

  // Naming conventions help navigation and avoid duplication
  // An Authority is named as VERB_SUBJECT_ENTITY
  // Verbs should initially follow CRUD (CREATE, RETRIEVE, UPDATE, DELETE)
  // Subjects are based on outward looking relationships (OWN, OTHERS)
  // Entities are based on primary entities (USER, CUSTOMER, CART, ITEM, INVOICE)

  // Roles (act as EnumSets from the fine grained authorities defined later)
  // Internal roles
  /**
   * The administrator role that can reach administration API functions
   */
  ROLE_ADMIN,
  /**
   * Reserved for client applications when making upstream server calls
   */
  ROLE_CLIENT,

  // User administration
  CREATE_USERS,
  RETRIEVE_USERS,
  UPDATE_USERS,
  DELETE_USERS,

  CHANGE_OWN_PASSWORD,
  // And so on
}
```

This is then added to an annotation like so:

```java
@POST
@Timed
@Path("/user")
public Response create(
  @RestrictedTo({Authority.ROLE_ADMIN})
  User adminUser,
  AdminCreateUserRequest createUserRequest) {
  // User must have authenticated to be here
}
```

Note that the `Authority` enum is merely a list of values. These values can be used in the database to create the usual User/Role/Authority structure so that new roles can be dynamically created that combine different Authority values as required. The `@RestrictedTo` annotation takes an array of Authority so that multiple Authority values can be passed to the authenticator.

### How do I write this annotation?

The code to create the annotation itself is pretty straightforward:

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.PARAMETER})
public @interface RestrictedTo {
  // No value assumes only an admin can reach the resource
  Authority[] value() default Authority.ROLE_ADMIN;
}
```

### And what backs the annotation?

The `InjectableProvider` handles the the process of associating the annotation with Jersey and providing the necessary component scope. It links the annotation to the authenticator.

```java
public class HmacServerRestrictedToProvider<T> implements InjectableProvider<RestrictedTo, Parameter> {
  static final Log LOG = Log.forClass(HmacServerRestrictedToProvider.class);

  private final Authenticator<HmacServerCredentials, T> authenticator;
  private final String realm;

  /**
   * Creates a new {@link HmacServerRestrictedToProvider} with the given {@link com.yammer.dropwizard.auth.Authenticator} and realm.
   *
   * @param authenticator the authenticator which will take the {@link HmacServerCredentials} and
   *                      convert them into instances of {@code T}
   * @param realm         the name of the authentication realm
   */
  public HmacServerRestrictedToProvider(Authenticator<HmacServerCredentials, T> authenticator, String realm) {
    this.authenticator = authenticator;
    this.realm = realm;
  }

  @Override
  public ComponentScope getScope() {
    return ComponentScope.PerRequest;
  }

  @Override
  public Injectable<?> getInjectable(ComponentContext ic,
                                     RestrictedTo a,
                                     Parameter c) {
    return new HmacServerRestrictedToInjectable<T>(authenticator, realm, a.value());
  }
}
```

### And what handles the call to the authenticator?

The `Injectable`. This provides access to the Jersey `HttpContext` and on to enable values to be read from the request. When Jersey processes access to the method it constructs the `Injectable` (and provides the required authorities) and then calls the `getValue()` method so that an object of type `T` can be injected. It is at this point that the authentication takes place:

```java
class HmacServerRestrictedToInjectable<T> extends AbstractHttpContextInjectable<T> {

  private final Authenticator<HmacServerCredentials, T> authenticator;
  private final String realm;
  private final Authority[] requiredAuthorities;

  /**
   * @param authenticator The Authenticator that will compare credentials
   * @param realm The authentication realm
   * @param requiredAuthorities The required authorities as provided by the RestrictedTo annotation
   */
  HmacServerRestrictedToInjectable(
    Authenticator<HmacServerCredentials, T> authenticator,
    String realm,
    Authority[] requiredAuthorities) {
    this.authenticator = authenticator;
    this.realm = realm;
    this.requiredAuthorities = requiredAuthorities;
  }

  @Override
  public T getValue(HttpContext httpContext) {

    try {

      // Get the Authorization header
      final String header = httpContext.getRequest().getHeaderValue(HttpHeaders.AUTHORIZATION);
      if (header != null) {

        // Expect form of "Authorization: <Algorithm> <ApiKey> <Signature>"
        final String[] authTokens = header.split(" ");

        if (authTokens.length != 3) {
          // Malformed
          HmacServerRestrictedToProvider.LOG.debug("Error decoding credentials (length is {})", authTokens.length);
          throw new WebApplicationException(Response.Status.BAD_REQUEST);
        }

        final String apiKey = authTokens[1];
        final String signature = authTokens[2];
        final ContainerRequest containerRequest = (ContainerRequest) httpContext.getRequest();

        // Build the canonical representation for the server side
        final String canonicalRepresentation = HmacUtils.createCanonicalRepresentation(containerRequest);
        LOG.debug("Server side canonical representation: '{}'",canonicalRepresentation);

        // Ignoring the offered <Algorithm> but could easily be supported
        final HmacServerCredentials credentials = new HmacServerCredentials("HmacSHA1", apiKey, signature, canonicalRepresentation, requiredAuthorities);

        final Optional<T> result = authenticator.authenticate(credentials);
        if (result.isPresent()) {
          return result.get();
        }
      }
    } catch (IllegalArgumentException e) {
      HmacServerRestrictedToProvider.LOG.debug(e, "Error decoding credentials");
    } catch (AuthenticationException e) {
      HmacServerRestrictedToProvider.LOG.warn(e, "Error authenticating credentials");
      throw new WebApplicationException(Response.Status.INTERNAL_SERVER_ERROR);
    }

    // Must have failed to be here
    throw new WebApplicationException(Response.status(Response.Status.UNAUTHORIZED)
      .header(HttpHeaders.AUTHORIZATION,
        String.format(HmacUtils.HEADER_VALUE, realm))
      .entity("Credentials are required to access this resource.")
      .type(MediaType.TEXT_PLAIN_TYPE)
      .build());
  }

}
```

The above is probably not perfect, but is enough to do the job. For the latest version, please refer to the MBM project.

### So what does the authenticator look like?

Just a normal credential checker, with one interesting subtlety that I’ll explain in a moment. A user is looked up from a persistent store somewhere using the offered API key. This yields the secret key which is appended to the canonical representation built earlier. The signature is verified and if they match then the user is authenticated, otherwise they are considered absent which represents an authentication failure.

There is an important security point to be made here regarding potential timing attacks. This was highlighted by Code Hale who has written an [extensive article on the subject][6]. In a nutshell, if you perform a fail-fast comparison of the signature strings then you reveal information to an attacker about how close to the correct value they are. This is because it will take longer to compare a string that nearly matches (e.g. “abcdefgh” against “abcdexyz”) than it will for one that fails at the first character (e.g. “qwerty” against “abcdexyz”). Although this difference is in the order of microseconds, a statistical picture can be built up through a large number of requests that slowly allows an attacker to break the secret key.

Releases of the JVM up to 1.6_16 had a fail-fast implementation in the `MessageDigest` code which had this flaw. It appears to have been fixed in 1.6_17+, but if you can’t be certain of the target JVM for your code then you should use your own. The code is below:

```java
@Component
public class HmacServerAuthenticator implements Authenticator<HmacServerCredentials, User> {

  @Resource(name="hibernateUserDao")
  private UserDao userDao;

  @Override
  public Optional<User> authenticate(HmacServerCredentials credentials) throws AuthenticationException {

    // Get the User referred to by the API key
    Optional<User> user = userDao.getByApiKey(credentials.getApiKey());
    if (!user.isPresent()) {
      return Optional.absent();
    }

    // Check that their authorities match their credentials
    if (!user.get().hasAllAuthorities(credentials.getRequiredAuthorities())) {
      return Optional.absent();
    }

    // Locate their secret key
    String secretKey = user.get().getSecretKey();

    String computedSignature = new String(
      HmacUtils.computeSignature(
        credentials.getAlgorithm(),
        credentials.getCanonicalRepresentation().getBytes(),
        secretKey.getBytes()));

    // Avoid timing attacks by verifying every byte every time
    if (isEqual(computedSignature.getBytes(), credentials.getDigest().getBytes())) {
      return user;
    }

    return Optional.absent();

  }

  /**
   * Performs a byte array comparison with a constant time
   *
   * @param a A byte array
   * @param b Another byte array
   * @return True if the byte array have equal contents
   */
  public static boolean isEqual(byte[] a, byte[] b) {
    if (a.length != b.length) {
      return false;
    }

    int result = 0;
    for (int i = 0; i < a.length; i++) {
      result |= a[i] ^ b[i];
    }
    return result == 0;
  }

}
```

### How do I plug this in to Dropwizard?

It’s all done during the service startup, and is just a few lines of code:

```java
// Configure the authenticator
HmacServerAuthenticator hmacAuthenticator = new HmacServerAuthenticator();
hmacAuthenticator.setUserDao(userDao);
CachingAuthenticator<HmacServerCredentials, User> cachingAuthenticator = CachingAuthenticator
  .wrap(hmacAuthenticator, CacheBuilderSpec.parse("maximumSize=10000, expireAfterAccess=10m"));

// Providers
environment.addProvider(new HmacServerRestrictedToProvider<User>(cachingAuthenticator, "REST"));
```

The `userDao` is assumed to have been created and configured earlier. Often DAOs are created and configured to use Hibernate through a Spring context in which case the following code could be used during initialisation to allow Spring beans with Dropwizard:

```java
// Start Spring context based on the provided location
ApplicationContext context = new ClassPathXmlApplicationContext(new String[]{
  "/spring/mbm-context.xml"
});

// Configure authenticator
HmacServerAuthenticator hmacAuthenticator = context.getBean(HmacServerAuthenticator.class);
CachingAuthenticator<HmacServerCredentials, User> cachingAuthenticator = CachingAuthenticator
  .wrap(hmacAuthenticator, CacheBuilderSpec.parse(configuration.getAuthenticationCachePolicy()));

// Providers
environment.addProvider(new HmacServerRestrictedToProvider<User>(cachingAuthenticator, "REST"));
```

### You’ve covered the server side, what about the client?

One of the great features of Dropwizard and the Jersey client is the ease with with RESTful endpoints can be tested in isolation. This greatly simplifies the process of functional testing. The representation output from the resources can be compared against test fixtures to ensure that it continues to meet requirements. However, there is a little bit of extra work that needs to be done in order to test an endpoint that requires authentication.

Jersey provides a useful client-side hook to allow final modification of a request before it is committed. By extending the `ClientFilter` class and adding it to the Jersey startup configuration we can introduce all the HMAC client side code and make it work with production client code as well. Here’s some code that does the job:

```java
public class HmacClientFilter extends ClientFilter {

  public static final String MBM_API_KEY = "mbm_api_key";
  public static final String MBM_SECRET_KEY = "mbm_secret_key";

  private static final Log LOG = Log.forClass(HmacClientFilter.class);

  private final Providers providers;

  public HmacClientFilter(Providers providers) {
    this.providers = providers;
  }

  public ClientResponse handle(ClientRequest cr) {
    ClientRequest mcr = modifyRequest(cr);

    // Call the next client handler in the filter chain
    ClientResponse resp = getNext().handle(mcr);

    // Modify the response
    return modifyResponse(resp);
  }

  /**
   * Handles the process of modifying the outbound request with suitable HMAC headers
   *
   * @param clientRequest The original client request
   * @return The modified client request
   */
  private ClientRequest modifyRequest(ClientRequest clientRequest) {

    // Check for modifications to the API key through properties
    Map<String, Object> properties = clientRequest.getProperties();
    if (properties == null) {
      throw new IllegalStateException("Client request properties are null");
    }
    if (!properties.containsKey(MBM_API_KEY) ) {
      throw new IllegalStateException("Client request '"+ MBM_API_KEY +"' is null");
    }
    if (!properties.containsKey(MBM_SECRET_KEY) ) {
      throw new IllegalStateException("Client request '"+ MBM_SECRET_KEY +"' is null");
    }
    String publicKey = properties.get(MBM_API_KEY).toString();
    String sharedSecret = properties.get(MBM_SECRET_KEY).toString();

    // Provide a short TTL
    String httpNow = DateUtils.formatHttpDateHeader(DateUtils.nowUtc().plusSeconds(5));
    clientRequest.getHeaders().put(HmacUtils.X_HMAC_DATE, Lists.<Object>newArrayList(httpNow));

    String canonicalRepresentation = HmacUtils.createCanonicalRepresentation(clientRequest,providers);
    LOG.debug("Client side canonical representation: '{}'", canonicalRepresentation);

    // Build the authorization header
    String signature = new String(HmacUtils.computeSignature("HmacSHA1", canonicalRepresentation.getBytes(), sharedSecret.getBytes()));
    String authorization = "HMAC " + publicKey + " " + signature;
    clientRequest.getHeaders().put(HttpHeaders.AUTHORIZATION, Lists.<Object>newArrayList(authorization));

    String curlCommand = HmacUtils.createCurlCommand(clientRequest,providers, canonicalRepresentation, sharedSecret, publicKey);
    LOG.debug("Client side curl command: '{}'", curlCommand);

    return clientRequest;
  }

  private ClientResponse modifyResponse(ClientResponse resp) {
    // Placeholder to complete the implementation
    return resp;
  }

}
```

There are a couple of points to note in the above code. The first is the use of the `clientRequest.getProperties()` to allow request-specific parameters to be passed in from external code. This is useful when we want to have a default set of credentials for when the client app talks to the server app, and to allow an individual user to step in and use theirs instead.

A second point is the use of the [curl command][7] provided in the debug log. This is a very handy debug tool when working with distributed systems since it allows you to mimic the behaviour of the application using an alternative command line based tool. It provides a quick way to diagnose problems and helps to focus attention on the problem at hand.

In order to bind the `ClientFilter` to the Jersey test configuration you simply need to add it at the end:

```java
@Before
  public void setUpJersey() throws Exception {

    // Provide any specialised resource configuration
    setUpResources();

    this.test = new JerseyTest() {
      @Override
      protected AppDescriptor configure() {
        final DropwizardResourceConfig config = new DropwizardResourceConfig(true);
        // Default singletons
        for (Object provider : JavaBundle.DEFAULT_PROVIDERS) {
          config.getSingletons().add(provider);
        }
        // Configure Jackson
        final Json json = getJson();
        config.getSingletons().add(new JacksonMessageBodyProvider(json));
        config.getSingletons().addAll(singletons);

        // Add providers
        for (Class<?> provider : providers) {
          config.getClasses().add(provider);
        }
        config.getClasses().add(OptionalQueryParamInjectableProvider.class);

        // Add any features (see FeaturesAndProperties)
        for (Map.Entry<String, Boolean> feature : features.entrySet()) {
          config.getFeatures().put(feature.getKey(), feature.getValue());
        }

        return new LowLevelAppDescriptor.Builder(config).build();
      }
    };

    // Allow final client request filtering for HMAC authentication
    // (this allows for secure tests)
    test.client().addFilter(new HmacClientFilter(
      test.client().getProviders()
    )
    );

    test.setUp();

  }
}
```

In order to get the above code to work, I had to create an alternative version of the Dropwizard `ResourceTest` which allowed for much greater customisation. The code for this alternative implementation is [all available in the MBM codebase][8].

### In conclusion

Hopefully this rather lengthy post will help you implement your own HMAC authentication scheme. Do bear in mind that OAuth2 may gain better support within Dropwizard and could end up being the better way to go. However, with the above and reference to the MBM codebase, you should be able to get your own version up and running much more easily.

Since you’ve taken the trouble to get all the way to the end (thanks!) you may want to consider some other authentication schemes more suited to browser interaction such as web form and session cookie authentication. I’ll be covering those in Dropwizard in another article.

 [1]: https://twitter.com/share
 [2]: https://github.com/gary-rowe/MultiBitMerchant
 [3]: http://en.wikipedia.org/wiki/Man-in-the-middle_attack
 [4]: http://hueniverse.com/2012/07/oauth-2-0-and-the-road-to-hell/
 [5]: https://github.com/gary-rowe/MultiBitMerchant/blob/develop/common/mbm-client/src/main/java/org/multibit/mbm/auth/hmac/HmacUtils.java
 [6]: http://codahale.com/a-lesson-in-timing-attacks/
 [7]: http://curl.haxx.se/
 [8]: https://github.com/gary-rowe/MultiBitMerchant/tree/develop/mbm/src/test/java/org/multibit/mbm/test