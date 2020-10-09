---
author: Gary Rowe
title: Stored procedures vs Hibernate
layout: post
tags:
  - Database
  - Design
  - Hibernate
  - SQL
redirect_from: /agilestack/2010/01/26/stored-procedures-vs-hibernate/
---

In this article I aim to examine whether or not stored procedures should be used instead of [Hibernate][2] based queries. I’ll assume that both Java developers and DBAs are likely to be reading this, so here is a quick glossary of common design patterns found in Hibernate applications:

Data Transfer Objects (DTOs) are used to transfer state information between layers in an application. They tend to represent the business domain and it’s relationships quite closely, and contain mapping information so that they can be persisted within the database.

Data Access Objects (DAOs) are used to perform all persistence operations: creating, retrieving, updating and deleting data from the database by means of manipulating DTOs. If a DTO is deleted, this is mapped to a SQL delete operation on the database.

Object/Relational Mapping (ORM) is the process by which application business state is transferred into relational data.

### Assumptions

The issue of whether to go wholly for a [Hibernate][2] based query approach, or, on the other side of the coin, a wholly stored procedure one is highly contentious with many strong arguments being presented for and against either approach. So let’s start by introducing some important assumptions:

1) both developers and DBAs are working for the same company and so are attempting to achieve the same objective of a simple to use process that promotes good practice;

2) the choice of RDBMS is not going to change in the near future so database independence is not an issue;

3) any move away from an entirely stored procedure approach will remove the burden of data consistency from the DBAs and place it on the application development team (programmers with DBAs acting as advisors);

4) the issue of “dynamic SQL” where SQL statements are created on the fly by, for example, concatenating strings without using parameters is not acceptable under any circumstances (do I hear SQL injection attack?) so all SQL generated within the application can be assumed to be parameterised;

5) application generated SQL is assumed to have come from an O/R mapping solution, such as Hibernate, not directly coded into the application;

6) long running queries acting on millions of rows are expected to be run as stored procedures since the network cost of moving the data over the wire would outweigh any performance improvement available through highly optimized application code.

Overall, there are only 3 choices available: persistence entirely through stored procedures; entirely through Hibernate; or a hybrid approach. Within the hybrid approach there is only the degree to which stored procedures are used within the application.

I have found that here are many myths and arguments associated with the stored procedure vs application SQL debate which collapse under close scrutiny. Here are a few (I’m sure that there are more):

### 1) Debugging stored procedures is hard

Many developers become used to working within a single IDE, and feel uncomfortable when forced to move outside it. In the case of stored procedures, it is frequently necessary for a Java developer to make use of an application such as Toad or JDeveloper to pick their way through a difficult SQL statement. To use this as an excuse not to use stored procedures is just laziness. The tools are there – learn to use them.

### 2) Stored procedures improve security

By only permitting data retrieval through stored procedures the impact of inappropriate or erroneous queries against the tables is greatly reduced leading to improved data security. Hmm. The only security that stored procedures provides is through the login credentials of the connecting user. If the application can delete users, then the connecting user can do so, if only by calling the stored procedure instead of typing in the SQL.

### 3) Stored procedures are faster than application SQL

While stored procedures are often considered to be the equivalent of assembly language for an RDBMS, the perceived performance gain is not always met. Almost all RDBMS implementations provide a caching mechanism such that the first time a query is encountered it incurs a performance penalty while it is parsed and compiled. Subsequent calls can immediately take advantage of the pre-compiled version and it doesn’t matter if this comes from within the RDBMS or the calling application. There are some very clever people behind these compilers and they will create a near optimal SQL statement. I’ll assume that the network lag for actually transmitting the query is assumed to be negligible.

For completeness, the case of stored procedures for update routines should be mentioned. An application can easily generate optimal update SQL for a single field involving a single parameter. In contrast, a stored procedure implementation will normally end up having to provide an “all fields” update mechanism with nullable parameters. This in turn leads to null checking logic within the stored procedure that is technically unnecessary.

Also important is that a stored procedure can take advantage of specialized functions available in the database that a generalized mapping solution, such as Hibernate, cannot emulate. There is, of course, the possibility of hard coded native SQL in the application but this should be frowned upon.

### 4) Stored procedures provide a single point of update

Given that all retrieval operations come from stored procedures, then any change to the implementation of those stored procedures is conducted behind the scenes and does not affect the applications that depend on it. Thus exclusive use of stored procedures introduces a single point of update should refactoring need to take place.

I agree that this is true up to a point, but consider the situation when an additional column is added to a table that needs to be included in the application. All stored procedures will need updating to include the new field, which will probably involve changing the signature of the stored procedure. This in turn will require that code in the application must be changed to support the new signature (even if the application does not make use of the new field).

Clearly this update process can be optimized, perhaps through versioned stored procedures or overloading with additional parameters, but the application is never completely isolated from the database. Over the course of time a large number of variants on the same theme may appear if multiple applications are served through the same API.

ORM frameworks like Hibernate overcome this by only including the required fields in their queries and relying on nullable fields being present. This helps to decouple them from changes to the underlying resultsets due to additional fields.

### 5) Data remains consistent because of stored procedures

A common argument for the use of stored procedures maintains that there is a guarantee that the data will always be consistent. This comes about due to a misunderstanding in the use of the term consistency. Consider the case when an application calls a sequence of stored procedures in order to fulfill a business process, but due to a coding error the last one is missed out.

The database is consistent from a mathematical point of view, but from a semantic point of view it is incomplete. If this sequence of calls was entirely within the application then only the developers can be blamed for the semantic error, whereas by forcing stored procedures then it is difficult to apportion blame – the database didn’t complain so it must be right.

In response, the DBA will necessarily have to include business logic in the stored procedures to enforce semantic consistency which negates the reasoning behind using stored procedures in the first place.

### 6) Stored procedures are application neutral

Once the API is written for the schema then any application can make use of them. Queries buried within JAR files tend to bind an application to a particular object model that may not be suitable.

This can be overcome by ensuring that application queries do not stray outside of the persistence layer, and that the persistence layer is designed against the specific requirements of the application which uses it. Reusable DTOs can then be ported to a common persistence framework but this does lead to a fair amount of complexity within the persistence layer dependency tree. I’d suggest using dependency management frameworks such as Maven or Ivy which can manage this transparently for you.

In order to make this approach work in the pure stored procedure approach, it is necessary to ensure that no knowledge of the database schema is present within the application. Table names should not be available, and columns and their types should be mapped internally to arbitrary names that are presented only through resultsets from the stored procedures. Therefore the application targets an abstract model presented by the stored procedure API which will completely decouple the application from any changes made behind the scenes.

Clearly, the creation of such an abstract model, in addition to that of the schema, is no easy task and the benefits it brings may be questionable. Is it really worth the effort?

### 7) Stored procedures make it obvious where the SQL is

Stored procedures benefit from easy and obvious navigation – they are in the database API packages. In addition, all databases provide a wealth of tools to both profile and subsequently optimize the queries as part of the development process, rather than as an afterthought when performance degradation is noticed.

In the application, the actual SQL being run is buried in the depths of the mapping framework and usually only appears in the debug logs. The location in the application code where this query is constructed is not always clear – it should be in the DAOs but can be hidden in annotations on DTOs. Consequently, it takes some time and skill to identify where the query is being constructed, and how to alter the structure to make it more efficient. However, by following well known design patterns for entity mapping, these inefficiencies can be ironed out early and permanently.

### 8) Hibernate is slow because it makes unnecessary calls

Hibernate can be made slow through the inappropriate use of eager fetching strategies where unnecessary associations are explored. This is known as [the Dredge anti-pattern][3]. If Hibernate is configured to only use lazy fetching then it can provide the optimal solution for data retrieval, outperforming stored procedures in some cases.

This bold statement needs an example to back it up. Consider the situation of a 3 way relationship between User, Role and Authority. The requirement is to fetch all Users with a first name of “Bob” (a set of 10 Users), along with all their Roles and Authorities. Hibernate can build this object graph in 3 queries: one for each entity followed by application side merging.

In contrast, a pure stored procedure approach will require a routine to filter on the User table to get the ones with the correct first name. Then the Role fetch routine will have to be called to get the Roles for each of the Users, then the Authorities fetch routine will have to be called for each of the Roles. This could lead to a very large number of procedure calls.

Clearly, an alternative would be a single procedure for that specific purpose that returns a union of the tables with inner joins as required, and an assortment of parameters for the filtering. Even then, the resultset will still have to be mapped so the gain from using a stored procedure becomes questionable, especially when the equivalent Hibernate query is 2 lines of obvious code.

### 9) Hibernate is slow because the SQL generated by the Criteria interface is not consistent

It has been said that Hibernate can also incur a performance hit if all queries are constructed through the Criteria interface instead of directly in HQL. The argument posits that this is because every time the query builder code is executed, say in a DAO, Hibernate will generate new alias names for the tables in the query. In Oracle this means that every time a new Criteria-based query is run, the database must create a QEP – query execution plan – as it is unable to match the SQL it has been given to any in its cache. Creating the QEP can take 30% of the time it takes for Oracle to respond to a SQL statement, so for the second and subsequent executions of the same (but for alias names) SQL statement, Criteria has a built-in overhead that makes it 50% slower than direct HQL.

This is no longer the case with Hibernate 3.3 and above. If it was ever true at all is in doubt since the Hibernate team would certainly aim to create optimal SQL wherever possible. Independent tests demonstrate that the same query is generated by the Criteria interface after repeated calls spanning transactions which is the equivalent of running the application under load. In each case the query remained identical and was therefore able to be cached by Oracle.

There is one grain of truth, though, in that it is necessary to create the query every time using the Criteria interface, whereas using named queries defined in HQL allows for precompilation during application start-up. However, this needs some perspective. The time taken to create a simple “between” query using the Criteria interface is approximately 3ms on an average PC. Embedding HQL within the application is not a good alternative since it does not lead to an intuitive mechanism for maintaining queries with differing fetching strategies, and so the Criteria based approach is deemed the better of the two.

### 10) Hibernate is difficult to learn

OK, now we’re getting desperate. Constructing a good persistence layer is difficult, regardless of the underlying ORM framework being used. Hibernate provides a wealth of useful tools to reduce the complexity of the resulting layer. In addition there are many examples of how to use Hibernate to solve various persistence problems. Examples include mapping a many to many relationship with attributes; creating an object hierarchy within a single table using discriminators; and managing lazy loading scenarios outside of the persistence layer to name but a few.  
Given a sufficient set of examples and a good ER diagram, any reasonable developer should be able to map out DAOs and DTOs for an arbitrary schema in a matter of hours.

### Conclusions

Hibernate should be default choice for persistence operations.  
When it comes to mapping and then persisting large graphs of DTOs the amount of code that needs to be written is minimal, frequently only requiring a single line of code. Careful construction of DAOs and DTOs using Java generics to enforce type safety further reinforces what is already an excellent persistence framework. The vast majority of work will be conducted by developers in the application domain, using object oriented programming techniques, so it makes sense to provide them with an object persistence model that reduces the complexity of the application.

 [1]: https://twitter.com/share
 [2]: https://www.hibernate.org/
 [3]: http://www.devx.com/Java/Article/29162/1954