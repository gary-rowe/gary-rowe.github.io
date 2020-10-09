---
author: Gary Rowe
title: How to build a Builder
layout: post
tags:
  - Design
  - HowTo
  - Principles
redirect_from: /agilestack/2012/07/11/how-to-build-a-builder/
---

Whenever I start to write out some domain object I always try to remind myself of two important principles:

1.  Avoid [anaemic domain objects][2]
2.  Build a Builder

The first principle may come as a surprise to some. After all, there are plenty of examples of JPA or JAXB annotated Java classes that simply consist of a bunch of getters and setters leaving all the complex business logic to Data Access Objects (DAOs). 

These are really just Data Transfer Objects (DTOs) rather than domain objects. A proper domain object contains more than just getters and setters, it has meaningful method names tied to appropriate business logic. Sure, it’ll have some annotations to assist with persistence, but on the whole it’ll be a fluent representation of the business domain in which it resides.

The second principle can seem like overkill to many since a domain object is such a trivial thing to create. However as [Bloch states in Effective Java][3]:

> “…the Builder pattern is a good choice when designing classes whose constructors or static factories would have more than a handful of parameters, especially if most of those parameters are optional.”

### So what makes a good Builder?

I have found that applying the following approach leads me to create a clean and consistent Builder interface:

*   Provide a Builder instance via a static method (`newInstance()` for example) – this allows a single chained statement to take place
*   Do not mutate the state of an underlying entity in response to configuration methods, instead record the state as a form of specification – this avoids the trap of the entity being in an inconsistent state
*   Provide a single `build()` method to perform the task of building, with subsequent calls to `build()` failing – just part of having a consistent API across patterns
*   Provide a consistent naming convention for configuration methods – I prefer to use a “with” prefix on these methods
*   Ensure that all configuration methods return the current instance of the Builder – this is to keep the chain going and promote a single line build

Applying the above gives rise to building code that looks like this:

```java
final User admin = UserBuilder
  .newInstance()
  .withUUID("trent123")
  .withUsername("trent")
  .withPassword("trent1")
  .withContactMethod(ContactMethod.FIRST_NAME, "Trent")
  .withContactMethod(ContactMethod.EMAIL, "admin@example.org")
  .withRole(adminRole)
  .build();
```

Of course, there is a temptation to use the Builder to create highly complex default starting configurations, such as `.withAdministratorRolesAndAuthorities()`. I have found that it is best to keep the focus tightly on the underlying domain object, and to use other dedicated Builders for the additional objects.

### Show me the code!

In my [MultiBit Merchant project][4], I have a User entity. Here is its Builder. I put it in here in its entirety so that you can simply copy-paste-modify your own versions from this template. It’s quite long… 

```java
public class UserBuilder {
  // Store specification state here
  private String openId;
  private String uuid = UUID.randomUUID().toString();
  private String secretKey;
  private String username;
  private String password;
  private Customer customer;

  // Keep a collection of visitors
  private List<AddContactMethod> addContactMethods = Lists.newArrayList();
  private List<AddRole> addRoles = Lists.newArrayList();

  // Prevent repeat builds
  private boolean isBuilt = false;

  public static UserBuilder newInstance() {
    return new UserBuilder();
  }

  public User build() {
    validateState();

    // User is a DTO and so requires a default constructor
    User user = new User();

    user.setOpenId(openId);

    if (uuid == null) {
      throw new IllegalStateException("UUID cannot be null");
    }
    user.setUUID(uuid);

    user.setSecretKey(secretKey);
    user.setUsername(username);

    if (password != null) {
      // Digest the plain password
      String encryptedPassword = new StrongPasswordEncryptor().encryptPassword(password);
      user.setPassword(encryptedPassword);
    }

    // Bi-directional relationship
    if (customer != null) {
      user.setCustomer(customer);
      customer.setUser(user);
    }

    for (AddRole addRole : addRoles) {
      addRole.applyTo(user);
    }

    for (AddContactMethod addContactMethod : addContactMethods) {
      addContactMethod.applyTo(user);
    }

    isBuilt = true;

    return user;
  }

  private void validateState() {
    if (isBuilt) {
      throw new IllegalStateException("The entity has been built");
    }
  }

  public UserBuilder withOpenId(String openId) {
    this.openId = openId;
    return this;
  }

  public UserBuilder withUUID(String uuid) {
    this.uuid = uuid;
    return this;
  }

  public UserBuilder withSecretKey(String secretKey) {
    this.secretKey = secretKey;
    return this;
  }

  public UserBuilder withContactMethod(ContactMethod contactMethod, String detail) {

    addContactMethods.add(new AddContactMethod(contactMethod, detail));

    return this;
  }

  public UserBuilder withRole(Role role) {

    addRoles.add(new AddRole(role));
    return this;
  }

  public UserBuilder withRoles(List<Role> roles) {

    for (Role role : roles) {
      addRoles.add(new AddRole(role));
    }

    return this;
  }

  public UserBuilder withUsername(String username) {
    this.username = username;
    return this;
  }

  public UserBuilder withPassword(String password) {
    this.password = password;
    return this;
  }

  public UserBuilder withCustomer(Customer customer) {
    this.customer = customer;
    return this;
  }

  private class AddContactMethod {
    private final ContactMethod contactMethod;
    private final String detail;

    private AddContactMethod(ContactMethod contactMethod, String detail) {
      this.contactMethod = contactMethod;
      this.detail = detail;
    }

    void applyTo(User user) {
      ContactMethodDetail contactMethodDetail = new ContactMethodDetail();
      contactMethodDetail.setPrimaryDetail(detail);

      user.setContactMethodDetail(contactMethod, contactMethodDetail);

    }
  }

  private class AddRole {
    private final Role role;

    private AddRole(Role role) {
      this.role = role;
    }

    void applyTo(User user) {

      UserRole userRole = new UserRole();

      UserRole.UserRolePk userRolePk = new UserRole.UserRolePk();
      userRolePk.setUser(user);
      userRolePk.setRole(role);

      userRole.setPrimaryKey(userRolePk);

      user.getUserRoles().add(userRole);

    }
  }
}
```

### What’s with the inner classes?

The inner classes (`AddContactMethod` and `AddRole`) are part of a [Visitor pattern][5] that is used to store and then apply state. This provides a simple, but powerful, technique to configure collections.

Of course, the above is only a snapshot of the current state of the project – it is certain to evolve – but the principles behind it will remain.

 [1]: https://twitter.com/share
 [2]: http://martinfowler.com/bliki/AnemicDomainModel.html
 [3]: http://www.amazon.co.uk/Effective-Java-Series-Joshua-Bloch/dp/0201310058
 [4]: https://github.com/gary-rowe/MultiBitMerchant
 [5]: http://en.wikipedia.org/wiki/Visitor_pattern#Java_example