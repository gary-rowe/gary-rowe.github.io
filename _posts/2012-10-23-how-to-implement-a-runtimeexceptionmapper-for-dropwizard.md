---
author: Gary Rowe
title: How to implement a RuntimeExceptionMapper for Dropwizard
layout: post
tags:
  - HowTo
  - Design
  - Dropwizard
redirect_from: /agilestack/2012/10/23/how-to-implement-a-runtimeexceptionmapper-for-dropwizard/
---

One common problem that web developers face is the elegant handling of exceptions in their applications. You want to give the browser the correct response code (e.g. 404 or 500), but you also want to ensure that the user experience of that failure maintains the style of the overall site. The default [Dropwizard][2] behaviour is rather minimal in this area (by design) and so I thought I’d post up an implementation to act as a starting point for others to use:

```java
import com.yammer.dropwizard.logging.Log;
import org.multibit.store.views.PublicFreemarkerView;

import javax.ws.rs.WebApplicationException;
import javax.ws.rs.core.Response;
import javax.ws.rs.ext.ExceptionMapper;
import javax.ws.rs.ext.Provider;

/**
 * <p>Provider to provide the following to Jersey framework:</p>
 * <ul>
 * <li>Provision of general runtime exception to response mapping</li>
 * </ul>
 */
@Provider
public class RuntimeExceptionMapper implements ExceptionMapper<RuntimeException> {

  private static final Log LOG = Log.forClass(RuntimeExceptionMapper.class);

  @Override
  public Response toResponse(RuntimeException runtime) {

    // Build default response
    Response defaultResponse = Response
      .serverError()
      .entity(new PublicFreemarkerView("error/500.ftl"))
      .build();

    // Check for any specific handling
    if (runtime instanceof WebApplicationException) {

      return handleWebApplicationException(runtime, defaultResponse);
    }

    // Use the default
    LOG.error(runtime, runtime.getMessage());
    return defaultResponse;

  }

  private Response handleWebApplicationException(RuntimeException exception, Response defaultResponse) {
    WebApplicationException webAppException = (WebApplicationException) exception;

    // No logging
    if (webAppException.getResponse().getStatus() == 401) {
      return Response
        .status(Response.Status.UNAUTHORIZED)
        .entity(new PublicFreemarkerView("error/401.ftl"))
        .build();
    }
    if (webAppException.getResponse().getStatus() == 404) {
      return Response
        .status(Response.Status.NOT_FOUND)
        .entity(new PublicFreemarkerView("error/404.ftl"))
        .build();
    }

    // Debug logging

    // Warn logging

    // Error logging
    LOG.error(exception, exception.getMessage());

    return defaultResponse;
  }

}
```
    

The response contains an entity (an instance of `PublicFreemarkerView`) which is simply a `View` that references a Freemarker template stored under `src/main/resources/assets/views/ftl`. The code is trivial, but I’ll include it here for completeness:

```java
public class PublicFreemarkerView extends View {

  public PublicFreemarkerView(String templateName) {
    super("/views/ftl/"+templateName);
  }

}
```
    

By adding the above (or including it in the scan) as a provider during startup you can rely on all your accidental `RuntimeException`s getting a decent presentation.

 [1]: https://twitter.com/share
 [2]: http://dropwizard.codahale.com/