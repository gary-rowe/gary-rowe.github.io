---
author: Gary Rowe
title: How to create a deterministic JAR
layout: post
tags:
  - HowTo
  - Maven
redirect_from: /agilestack/2013/08/08/how-to-create-a-deterministic-jar/
---
The other day I needed to verify that my machine had built a JAR file correctly by comparing it to another build on a
different machine. I noted the SHA1 hash of the original, built it again and the two were different. The source code was
identical since the git commit fingerprint was the same.

It turns out that JARs contain time-sensitive data that throws out the SHA1 signature.

What I needed was a deterministic JAR.

### What is a deterministic JAR?

This is a JAR that will build to a known SHA1 hash every time. It has all time- and build-sensitive information
stripped out of it leaving only the compiler as the variant. The Java specification does not mandate a particular
byte code output for a given source code input and it is possible for it to vary across operating systems,
however it does appear to be the case that for the same vendor, operating system and point release (e.g. 1.6.0_52) that
you can get a deterministic JAR.

### What do I have to do?

You'll have to abandon the standard JAR plugin that comes with Maven in favour of the assembly plugin. Also you must
supply your own MANIFEST.MF file with known values and finally you'll have to use the `touch` utility to set all assets
to a known timestamp.

We start with the assembly plugin configuration:

```xml
src/main/assembly/zip.xml:

<assembly xmlns="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.0 http://maven.apache.org/xsd/assembly-1.1.0.xsd">
  <id>deterministic</id>
  <baseDirectory>/</baseDirectory>
  <formats>
    <format>zip</format>
  </formats>
  <fileSets>
    <fileSet>
      <directory>${project.build.directory}/classes</directory>
      <outputDirectory>/</outputDirectory>
    </fileSet>
  </fileSets>
</assembly>
```

Next, you'll create your own MANIFEST.MF. You may wish to edit this example, but **make sure that you leave an extra
CRLF at the end of the last entry** or it will be invalid.

```text
src/main/resources/META-INF/MANIFEST.MF:

Manifest-Version: 1.0
Archiver-Version: Plexus Archiver
Created-By: Apache Maven
Built-By: bitcoinj
Build-Jdk: 1.6.0

```

Finally you need to add some details to the pom.xml. This comes in 3 stages. The first is to `touch` all the contents
 of the production code and set them to a known date. You might want to indicate the version number using the time
 field for fun. Next is the zipping process with the final stage renaming the artifact to a JAR for compatibility.

```xml
pom.xml:

<plugins>

... other plugins ...

    <!-- Step 1: Set all timestamps to same value -->
    <plugin>
    <artifactId>maven-antrun-plugin</artifactId>
    <version>1.7</version>
    <executions>
      <execution>
        <id>1-touch-classes</id>
        <phase>prepare-package</phase>
        <configuration>
          <target>
            <touch datetime="01/01/2000 00:10:00 am">
              <fileset dir="target/classes"/>
            </touch>
          </target>
        </configuration>
        <goals>
          <goal>run</goal>
        </goals>
      </execution>
    </executions>
    </plugin>

    <!-- Step 2: Assemble as a ZIP to avoid MANIFEST.MF timestamp -->
    <plugin>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>2.2.1</version>
    <configuration>
      <descriptors>
        <descriptor>src/main/assembly/zip.xml</descriptor>
      </descriptors>
    </configuration>
    <executions>
      <execution>
        <id>2-make-assembly</id>
        <phase>prepare-package</phase>
        <goals>
          <goal>single</goal>
        </goals>
      </execution>
    </executions>
    </plugin>

    <!-- Step 3: Rename ZIP as JAR -->
    <plugin>
    <artifactId>maven-antrun-plugin</artifactId>
    <version>1.7</version>
    <executions>
      <execution>
        <id>3-rename-assembly</id>
        <phase>package</phase>
        <configuration>
          <target>
            <move file="${project.build.directory}/${project.build.finalName}-deterministic.zip"
                  tofile="${project.build.directory}/${project.build.finalName}-deterministic.jar"/>
          </target>
        </configuration>
        <goals>
          <goal>run</goal>
        </goals>
      </execution>
    </executions>
    </plugin>

... more plugins ...

</plugins>
```

The above should result in a repeatable deterministic build for your project.

### But mine's different!

It may be that something has changed during the build process. Here's a script (works on OS X and is
easily modified for Linux) that will unpack a JAR and perform a SHA1 on each file within it and put the results into
a file (you should provide an index as a parameter to the script).

```bash
openssl sha1 target/example-1.0.0-SNAPSHOT.jar > sha1_$1_jar.txt
mkdir target/example-1.0.0-SNAPSHOT
cd target/example-1.0.0-SNAPSHOT
jar xf ../example-1.0.0-SNAPSHOT.jar
cd ../..
find ./target/example-1.0.0-SNAPSHOT -type f -print0 | xargs -0 openssl sha1 > sha1_$1_all.txt
```

If you build the JAR several times and run the script you can simply use `diff` to determine where the problem lies.

### Conclusion

It is possible to create a deterministic JAR but there are several hoops to jump through. The compiler will introduce
variations that are outside of your control and so it is necessary to have an identical compiler to that which
originally created the JAR (and the target SHA1).

If such information is available and the build is repeatable across environments then this can form the basis of a
secure artifact. As such developers may consider using *threshold signing* where several developers repeat the build,
get the required result and sign the original artifact. Once a certain number of signatures are acquired then the
artifact is considered safe for release.
