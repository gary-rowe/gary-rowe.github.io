---
author: Gary Rowe
title: Preventing Dependency Chain Attacks in Maven
layout: post
tags:
  - HowTo
  - Maven
redirect_from: /agilestack/2013/07/03/preventing-dependency-chain-attacks-in-maven/
---
As regular readers will be aware, I do a lot of work within the [Bitcoin community](http://bitcoin.org) contributing code wherever I can.
Recently an interesting problem came up with the [Bitcoinj](https://code.google.com/p/bitcoinj/) library which has
highlighted a particular weakness within Maven that I feel more developers should be aware of: the dependency-chain
attack.

### What is a dependency-chain attack?

Imagine that some malicious developer, Mallory, was able to gain write access to a Maven repository that you used as
part of your development. Mallory knows from examining your `pom.xml` that you depend on several artifacts which are
served from this repository and so he sets about creating a new version of one of these artifacts containing some
code that is designed to detect and disrupt your application. In the case of the Bitcoinj library this could be to
copy the unencrypted private keys back to his server.

Mallory deletes the original artifact and replaces it with his own which has the same version number as the original
but a different SHA1 and does not contain a valid signature.

How would you detect that in your Maven build?

The short answer is that you can't. The SHA1 is valid and Maven does not check JAR signatures during the build process.
You might get lucky in that you won't download the artifact since you have a local copy of the release, but if you move
to another machine then you'll be pulling down the malicious code.

You need a whitelist.

### A whitelist?

By obtaining a list of known good artifacts and their SHA1 signatures it is possible to build up a list of "approved"
artifacts and detect when any have changed from expectations. As long as you remain in control of your source code 
repository then you can be sure that your whitelist has not been tampered with. Obviously, if Mallory got control of
your source code then all bets are off.

I have released a simple `DigestRule` for use with the [Maven Enforcer Plugin](http://maven.apache.org/enforcer/maven-enforcer-plugin/)
to provide such a whitelist. You can choose how detailed you want it to be depending on your own security requirements,
but bear in mind that every unchecked dependency is a way in for an attacker through the back door.

### How is it configured?

The configuration below shows how you would use the `DigestRule` in your own projects. Of course the values used
below should be treated only as examples. 

In production you would get the list of URNs as a GPG signed configuration from the developers of the project that you 
were depending on. 

```xml
<build>
  <plugins>
    ...
      <!-- Use the Enforcer to verify build integrity -->
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-enforcer-plugin</artifactId>
        <version>1.2</version>
        <executions>
          <execution>
            <id>enforce</id>
            <phase>verify</phase>
            <goals>
              <goal>enforce</goal>
            </goals>
            <configuration>
              <rules>
                <digestRule implementation="uk.co.froot.maven.enforcer.DigestRule">

                  <!-- Create a snapshot to build the list of URNs below -->
                  <buildSnapshot>true</buildSnapshot>

                  <!-- List of required hashes -->
                  <!-- Format is URN of groupId:artifactId:version:type:classifier:scope:hash -->
                  <!-- classifier is "null" if not present -->
                  <urns>

                    <urn>antlr:antlr:2.7.7:jar:null:compile:83cd2cd674a217ade95a4bb83a8a14f351f48bd0</urn>
                    <urn>dom4j:dom4j:1.6.1:jar:null:compile:5d3ccc056b6f056dbf0dddfdf43894b9065a8f94</urn>
                    <urn>org.bouncycastle:bcprov-jdk15:1.46:jar:null:compile:d726ceb2dcc711ef066cc639c12d856128ea1ef1</urn>
                    <urn>org.hibernate.common:hibernate-commons-annotations:4.0.1.Final:jar:null:compile:78bcf608d997d0529be2f4f781fdc89e801c9e88</urn>
                    <urn>org.hibernate.javax.persistence:hibernate-jpa-2.0-api:1.0.1.Final:jar:null:compile:3306a165afa81938fc3d8a0948e891de9f6b192b</urn>
                    <urn>org.hibernate:hibernate-core:4.1.8.Final:jar:null:compile:82b420eaf9f34f94ed5295454b068e62a9a58320</urn>
                    <urn>org.hibernate:hibernate-entitymanager:4.1.8.Final:jar:null:compile:70a29cc959862b975647f9a03145274afb15fc3a</urn>
                    <urn>org.javassist:javassist:3.15.0-GA:jar:null:compile:79907309ca4bb4e5e51d4086cc4179b2611358d7</urn>
                    <urn>org.jboss.logging:jboss-logging:3.1.0.GA:jar:null:compile:c71f2856e7b60efe485db39b37a31811e6c84365</urn>
                    <urn>org.jboss.spec.javax.transaction:jboss-transaction-api_1.1_spec:1.0.0.Final:jar:null:compile:2ab6236535e085d86f37fd97ddfdd35c88c1a419</urn>

                    <!-- A check for the rules themselves -->
                    <urn>uk.co.froot.maven.enforcer:digest-enforcer-rules:0.0.1:jar:null:runtime:16a9e04f3fe4bb143c42782d07d5faf65b32106f</urn>

                  </urns>

                </digestRule>
              </rules>
            </configuration>
          </execution>
        </executions>

        <!-- Ensure we download the enforcer rules -->
        <dependencies>
          <dependency>
            <groupId>uk.co.froot.maven.enforcer</groupId>
            <artifactId>digest-enforcer-rules</artifactId>
            <version>0.0.1</version>
          </dependency>
        </dependencies>

      </plugin>
    ...
  </plugins>
</build>

```

### I'm not typing in all that!

Clearly trying to manually create the list of URNs would be a painful process, particularly in a large project with many
layers of transitive dependencies to explore. Fortunately, the `buildSnapshot` flag will cause the plugin to examine all
the resolved dependencies within your project and build a list of URNs that you can copy-paste (with caution) into your build.

### How do I use it?

For maximum effect, the rules should be triggered during the `verify` phase so that all the dependencies that could affect
the build will have been pulled in. This has the useful side effect that as a developer you're not continuously checking
yourself for every build - only when you're about to perform an `install` or `deploy`.

You may want to grep/find on a case-sensitive match for "URN" to find the verification messages.

You can try it out on itself by [checking out the source code from GitHub](https://github.com/gary-rowe/BitcoinjEnforcerRules):

```bash
$ mvn clean install
```

The reactor will first build the Digest Enforcer Rules and then go on to build another artifact that depends on them
working (the Rule Tester project). This second project demonstrates how you would include Digest Enforcer Rules in
your projects.

### Does it work with third-party build systems?

Yes. One of the design goals was to allow Bitcoinj to be deployed into Maven Central with sufficient support that any
compromise to either it or its supporting libraries could be detected. Now your projects that include Bitcoinj will be
able to build through Travis [or your own equivalent](http://gary-rowe.com/agilestack/2013/02/14/how-to-deploy-dynamic-sites-with-git/) (once Bitcoinj arrives in Maven Central or another controlled repo).

### How do I check the checker?

Trust has to begin somewhere so I'm going to provide some signed declarations for each version in the GitHub repo. These can be validated
against my public key [59A81D7B](http://pgp.mit.edu:11371/pks/lookup?op=get&search=0x2183BCD259A81D7B). Obviously, you can also compile this code yourself and obtain the same result.

Why not try to verify [the certificate for version 0.0.1](https://raw.github.com/gary-rowe/BitcoinjEnforcerRules/master/certificate-0.0.1.asc) for practice?

If you are at all concerned about dependency-chain attacks, or just want to be that little bit safer when building, then please
take a look at the project and [offer up some critique](https://github.com/gary-rowe/BitcoinjEnforcerRules/issues) if it doesn't meet your standards.

### This could save my department a fortune...

The project is not tied to Bitcoinj so developers in secure environments such as research, military or government establishments 
may also want to consider this as an additional layer of protection to those already in place.

It may even allow you to **start using open source where once you were unable** due to the dependency-chain risk saving your
department vast amounts of money otherwise spent on validation fees.
