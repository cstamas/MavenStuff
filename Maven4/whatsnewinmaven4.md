# What's new in Maven 4?

Maven is more than 20 years old and is one of the most used build-tool in the Java world.
Since all the years one important rule was the highest backward-compatibility as possible, especially with its [POM-schema with Model version 4.0.0][2], used for the build itself, but also by consumers.
This made Maven more than a tool, but a whole ecosystem with a lot of things depended on the POM, esp. the Maven Central repository, other build tools and IDEs.
But this stable schema comes at a price - the lack of flexibility.

> "With the Maven build schema preserved in amber, we can’t evolve much: we’ll stay forever with Maven 3 minor releases, unable to implement improvements that we imagine will require seriously updating the POM schema…"
> &mdash; <cite>[Hervé Boutemy][1]</cite>

Maven 4 will prepare to changes which are impossible nowadays, like a complete new build schema.
This article presents and explains major changes brought by Maven 4, grouped into several topics.

## POM-Changes

### Build-POM and Consumer-POM
As written in the introduction the Model version 4.0.0 is used not only for the build, but also by consumers of the artifact.
But several contents of the POM are only necessary for the build while others like the dependencies are also needed by the consumer.
Maven 4 will therefore differentiate between a "Build-POM" and a "Consumer-POM".
As names suggest the "Build-POM" will contain all information needed for to build the artifact, e.g. used plugins and their configuration, while the "Consumer-POM", which is created during the Maven build, will not contain those.
This POM will only keep those which are really needed to use the artifact, e.g. dependency information.


### Model version 4.1.0
With now having two types of POM Maven 4 can already make the additions to the Build-POM as it will only be used by Maven (and of course IDEs).
Therefor with Maven 4 a new Model version 4.1.0 is introduced.
This version introduces some new elements and attributs, while others got marked as deprecated. 

### Modules are now subprojects

From the early days of Maven 1 till today, all build information are stored in the POM, short for "Project Object Model".
Together with build folders and other files the wording "Maven project" is used.
However, project containing multiple parts, e.g. an API and a client, each of those parts was called "module" and listed in the `<modules>` section of the POM, leading to the terms of "multi-module project".
This wording introduced some inconsistency, esp. as projects with any `<modules>` section are often called "single-module".
Since the introduction of the Java Platform Module System in Java 9 the term "module" raised additional confusion

Maven 4 gets rid of this by naming "modules" as what they are - subprojects.
Model version 4.1.0 contains a new element `<subprojects>`analogous to the now deprecated, but still usable, element `<modules>`.




### Comparing Build-POM and Consumer-POM

The following table shows a rough comparision about which content is available in which POM type. 

| Content                                                 | Build-POM | Consumer-POM |
|:--------------------------------------------------------|:---------:|:------------:|
| Model version                                           |4.1.0|    4.0.0     |
| Full qualified parent subproject dependency information |❌|      ❌       |
| 3rd party dependency information                        |✅|      ✅       |











## Workflow changes





<!--- Sources -->

[1]: https://www.javaadvent.com/2021/12/from-maven-3-to-maven-5.html
[2]: https://maven.apache.org/pom.html