# What's new in Maven 4?

Maven is more than 20 years old and is one of the most used build-tool in the Java world.
Since all the years one important rule was the highest backward-compatibility as possible, especially with its [POM-schema with Model version 4.0.0][2], used for the build itself, but also by consumers.
This made Maven more than a tool, but a whole ecosystem with a lot of things depended on the POM, esp. the Maven Central repository, other build tools and IDEs.
But this stable schema comes at a price - the lack of flexibility.

> "With the Maven build schema preserved in amber, we can’t evolve much: we’ll stay forever with Maven 3 minor releases, unable to implement improvements that we imagine will require seriously updating the POM schema…"
> &mdash; <cite>[Hervé Boutemy (in Javaadvent 2021)][1]</cite>

Maven 4 will prepare to changes which are impossible nowadays, like a complete new build schema.

Another pain point of Maven 3 is a code base with a lot of "old", "dirty", non performant and duplicated code which costs the volunteers who maintain Maven a lot of time.
This not only means that the Maven codebases contains old Java code that can be optimized nowadays, but also old dependencies and bad API design of its own APIs, especially for Maven plugins.
Therefor Maven 4 will also be a maintenance release.   

This article presents and explains major changes brought by Maven 4, grouped into several topics.

## POM-Changes

### Build-POM and Consumer-POM
As written in the introduction the Model version 4.0.0 is used not only for the build, but also by consumers of the artifact.
But several contents of the POM are only necessary for the build while others like the dependencies are also needed by the consumer.
Maven 4 will therefore differentiate between a "Build-POM" and a "Consumer-POM".
As names suggest the "Build-POM" will contain all information needed for to build the artifact, e.g. used plugins and their configuration, while the "Consumer-POM", which is created during the Maven build, will not contain those.
This POM will only keep those which are really needed to use the artifact, e.g. dependency information.

**Note**: See below for a comparison of the content of both POMs. 


### Model version 4.1.0
With now having two types of POM Maven 4 can already make the additions to the Build-POM as it will only be used by Maven (and of course IDEs).
Therefor with Maven 4 a new Model version 4.1.0 is introduced.
This version introduces some new elements and attributs, while others got marked as deprecated.
To not break the ecosystem ist version is only available for the Build-POM, while the Consumer-POM will still use version 4.0.0.

### Modules are now subprojects

From the early days of Maven 1 till today, all build information are stored in the POM, short for "Project Object Model".
Together with build folders and other files the wording "Maven project" is used.
However, project containing multiple parts, e.g. an API and a client, each of those parts was called "module" and listed in the `<modules>` section of the POM, leading to the terms of "multi-module project".
This wording introduced some inconsistency, esp. as projects with any `<modules>` section are often called "single-module".
Since the introduction of the [Java Platform Module System][3] in Java 9 the term "module" raised additional confusion

Maven 4 gets rid of this by naming "modules" as what they are - subprojects.
Model version 4.1.0 contains a new element `<subprojects>`analogous to the now deprecated, but still usable, element `<modules>`.

### New packaging type: bom
Maven 4 introduces a dedicated packaging type to provide a [Bill of Material BOM][4] called "bom".
While the new type is only available with model Version 4.1.0 the final outcome is a full Maven 3 compatible (model 4.0.0) POM file!
See the link above for an example or the [live coding by Karl Heinz Marbaise at IntelliJ IDEA Conf 2004][5].

### Declaring the root project

TOOD





### Comparing Build-POM and Consumer-POM

The following table shows a rough comparison about which content is available in which POM type when using Maven 4.

**Notes**:
* The column "Consumer-POM" obviously does not apply for artefacts that are of type "pom" or "bom"!
* Some of the build-related content which is (as of now) still available in the Consumer-POM might be available only in the Build-POM in the future. 

| Content                                                 | Build-POM | Consumer-POM |
|:--------------------------------------------------------|:---------:|:------------:|
| Model version                                           |   4.1.0   |    4.0.0     |
| Full qualified parent subproject dependency information |     ❌     |      ❌       |
| 3rd party dependency information                        |     ✅     |      ✅       |
| Properties                                              |     ✅     |      ❌       |
| Plugin configuration                                    |     ✅     |      ❌       |
| Repository information                                  |     ✅     |      ✅       |
| Project information / environment settings              |     ✅     |      ✅       |
| Deployment to remote repository                         |     ✅     |      ✅       |


## Improvements for subprojects 

### Automatic versioning of parents and subprojects

### Further improvements

* Consistent timestamps
* Deploy all or none





## Workflow and runtime changes

### Java 17 required to run Maven
The required Java version to run Maven 4 will be Java 17 
This allows Maven (and its maintainers) to make use of newer language features and improvements brought by the JDK.


**Important note**: Java 17 will only be needed to **run Maven**!
You will still be able to compile against older Java versions using the same [compiler plugin configuration][6] as before!
If this does not fit your requirements, because you need to compile against or using another JDK, please have a look at the [Guide to Using Toolchains][7] (or the article [Introduction to Maven Toolchains][8] by Maven maintainer Maarten Mulders).

*Side information: The ballot about the required Java version was hold in March 2024, shortly before Java 22 was released.
Java 17 was chosen over Java 21, because it was (at this time) the second last Java version for which many vendors offer long-term-support.*

## "Fail on severity" parameter
Maven 4 introduces a "fail on severity" build parameter, which will break the build when at least one log message matches the given argument.

The parameter can either be used by its full name (`--fail-on-severity`) or as a short handle (`-fos`).
The parameter is followed by an argument of a log level severity, e.g. `WARN`.


## Maven plugins and dependencies

As written in the introduction, Maven 4 will contain huge code and API updates (not only because Java 17 language features can be used) and even removals, resulting in breaking very old Maven plugins, which were not updated to the recommended APIs.
For example the Plexus Containers dependency was removed - after being deprecated since Maven 3.2 (2010)!

If you are maintaining a Maven plugin, you should test it with Maven 3.9.x and have a close look at the warnings.



# Issue overview

The following table shows links to the issues of changes, mentioned here.
For a full list of all issues, please see the [Maven releases history][10], starting with Maven 4.0.0-alpha-2 (alpha-1 was skipped).


TODO.

| Topic           | JIRA-Issue |
|:----------------|:---------:|
|Build-POM and Consumer-POM|xx|
| Require Java 17 |[MNG-8061][9]|




<!--- Links -->

[1]: https://www.javaadvent.com/2021/12/from-maven-3-to-maven-5.html
[2]: https://maven.apache.org/pom.html
[3]: https://en.wikipedia.org/wiki/Java_Platform_Module_System
[4]: https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#bill-of-materials-bom-poms
[5]: https://www.youtube.com/watch?v=ZD_YxTmQ16Q&t=16710s
[6]: https://maven.apache.org/plugins/maven-compiler-plugin/examples/set-compiler-release.html
[7]: https://maven.apache.org/guides/mini/guide-using-toolchains.html
[8]: https://maarten.mulders.it/2021/03/introduction-to-maven-toolchains/
[9]: https://issues.apache.org/jira/projects/MNG/issues/MNG-8061
[10]: https://maven.apache.org/docs/history.html