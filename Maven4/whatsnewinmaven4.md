# What's new in Maven 4?

> **THIS IS A DRAFT AND THEREFOR WORK IN PROGRESS ARTICLE**

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
For an example see the link above or the [live coding by Maven maintainer Karl Heinz Marbaise at IntelliJ IDEA Conf 2004][5].

### Comparing Build-POM and Consumer-POM
The following table shows a rough comparison about which content is available in which POM type when using Maven 4.

**Notes**:
* The column "Consumer-POM" obviously does not apply for artefacts that are of type "pom" or "bom"!
* Some of the build-related content which is (as of now) still available in the Consumer-POM might be available only in the Build-POM in the future. 

| Content                                                              | Build-POM | Consumer-POM |
|:---------------------------------------------------------------------|:---------:|:------------:|
| Model version                                                        |   4.1.0   |    4.0.0     |
| 3rd party dependency information                                     |     ✅     |      ✅       |
| Properties                                                           |     ✅     |      ❌       |
| Plugin configuration                                                 |     ✅     |      ❌       |
| Repository information                                               |     ✅     |      ✅       |
| Project information / environment settings                           |     ✅     |      ✅       |
| Deployment to remote repository                                      |     ✅     |      ✅       |


### Declaring the root directory and directory variables
Everytime a Maven build is executed it has to determine the projects root so identify things like the parent project, directory information and so on.
To "help" Maven finding the root folder, you can create a `.mvn` folder in your root directory.
This folder is intended to contain project specific configuration to run Maven, e.g. a `maven.config` or `jvm.config` file, and therefore was also considered as the root folder.
With Maven 4 there is a second option to clearly define the root folder.
Model version 4.1.0, usable for the Build-POM, adds the boolean attribute called `root` in the `<project>` element.
When this attribute is set to true (default is false) the directory of this POM file is considered the root directory.

Another pain point in relation of the root directly is that until Maven 4 there was no official variable to make use of the root folder in your POM files, e.g. when you want to define the path to a `checkstyle-suppressions.xml` file for the checkstyle plugin.
Maven 4 now provides official variables to reference the root directory in your POM configuration.
The following table shows the official variables.

| Variable                   |  Scope  | Definition                                                              | Always |
|:---------------------------|:-------:|:------------------------------------------------------------------------|:------:|
| `${project.rootDirectory}` | Project | `.mvn` folder or `root` attribute in pom                                |   No   |
| `${session.topDirectory}`  | Session | Current directory or `--file` argument                                  |  Yes   |
| `${session.rootDirectory}` | Session | `.mvn` folder or `root` attribute in pom for the `topDirectory` project |   No   |

As you can see these variables differentiate by their scope, where `project` is always related to the Maven project's definition (you could interpret this as the POM files) and `session` is the actual execution of a maven build and is therefore related to the folder from where you start Maven.
As a consequence of the definition it's clear that the `rootDirectory` can only contain a value when either a `.mvn` folder is defined or the `root` attribute is set to true.
However, if defined it should always have the same value for a given project, whereas the value of the`topDirectory` variable can change depending on the execution point. 

Keep in mind that the root directory of the whole project (when considering multiple subprojects) is different from each subprojects' own base directory, which was and is still accessibly via the `${basedir}` property for the usage in POM files and will always have a value.

**Note:** In the past some people "hacked" workarounds for the`rootDirectory` variables, mostly by using internal variables.
Starting with Maven 4 those "hacks" will most probably not work anymore, because some of those variables were removed or at least marked as deprecated.
See JIRA issue [MNG-7038][15] and the related [Pull Request for MNG-7038][16] for more information.


## Improvements for subprojects

### Automatic versioning
Maven 4 finally ships one of the oldest improvement requests - automatic parent versioning ([MNG-624][17], created in July 2005 and originally planed for Maven 2)!
As expected it's no longer required to define the parent versions in each subproject, when using the new model version 4.1.0.
This is also extended to dependencies of project own subprojects and reduces the need to update POM files for new versions even more!

The following code snippet shows the parents and dependency definition without the version tag.
```xml
<project xmlns="http://maven.apache.org/POM/4.1.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.1.0 http://maven.apache.org/xsd/maven-4.1.0.xsd">
    <modelVersion>4.1.0</modelVersion>
    
    <parent>
      <groupId>my.parents.groupId</groupId>
      <artifactId>my.parents.artifactId</artifactId>
    </parent>
    
    <artifactId>myOwnSubprojectArtifactId</artifactId>
    
    <dependencies>
        <dependency>
         <groupId>the.dependent.subproject.groupId</groupId>
         <artifactId>the.dependent.subproject.artifactId</artifactId>
      </dependency>
    </dependencies>
</project>
```

### Full support of CI-friendly variables
Maven 3.5.0 introduced partly supported usage of CI-friendly variables, e.g. `${revision}`, in your POM files.
However, this still required the usage of the [Flatten Maven Plugin][20] for full functionality.
Since Maven 4 no additional plugin is needed anymore, but full build-in support is provided.
You can now use variables as versions in your configuration, e.g.

```xml
<groupId>my.groupId</groupId>
<artifactId>my.artifactId</artifactId>
<version>${revision}</version>
```
Of course, you have to provide a value for this variable when starting the build, for example by a `maven.config` file or as a parameter, e.g. `mvn verify -Drevision=4.0.1`, which is commonly done in CI pipelines. 

Maven maintainer Karl Heinz Marbaise shows a larger example in his [article "Maven 4 - Part I - Easier Versions"][21].

### Reactor improvements and fixes
Building a project with multiple subprojects could cause trouble when one subproject was dependent from one of the others and its own build failed for whatever reason.
Maven was telling the user to (fix the error and then) resume the build with `--resume-from :<nameOfTheFailingSubproject>`, which instantly fails the build again as the needed other subproject couldn't be found (as it was not rebuild too).
Using `--also-make :<nameOfTheDependentSubproject>` was no help in the past as it was ignored due the ma long-stand bug [MNG-6863][11] - which is finally fixed with Maven 4!
**So the "argument" to blindly use `mvn clean install` as a "workaround" for this (never intended) behavior is gone!
Don't use `mvn clean install`, but `mvn verify` for your regular builds!**
To improve usability when resuming a failed build you can now use `--resume` or its short parameter `-r` to resume a build from the subproject that last failed.
So you don't have to manually pass the name of the failed subproject as the starting point to resume from.
The reactor is now also aware of successfully build subprojects when the overall build failed, and will skip to rebuild those if you resume the build.  
With Maven 4 it's also aware of subfolder builds [MNG-6118][12], which becomes pretty handy when you only want to execute tools (e.g. Jetty) on/with certain subprojects, but not on every subproject.
See Maven maintainer Maarten Mulders's article ["What's new in maven 4" (2020)][13] for a small example.

### Further improvements
Further improvements to subprojects will also improve the daily work with those.
Thanks to [MNG-6754][14] all subprojects will now have consistent timestamps in there packaged archives, while in Maven 3 each subproject was having a different one.
This should make it easier to identify the archives which belong together.
When using Maven 3, deploying a project with multiple subprojects could end up in the situation where some (successfully build) subprojects where deployed to the (local or remote) repository, but failed subprojects were obviously not.
This was finally changed in Maven 4 to the way most users are expecting:
Only deploy when all subprojects were build successfully.


## Workflow and runtime changes

### Java 17 required to run Maven
The required Java version to run Maven 4 will be Java 17 
This allows Maven (and its maintainers) to make use of newer language features and improvements brought by the JDK.

**Important note**: Java 17 will only be needed to **run Maven**!
You will still be able to compile against older Java versions using the same [compiler plugin configuration][6] as before!
If this does not fit your requirements, because you need to compile against or using another JDK, please have a look at the [Guide to Using Toolchains][7] (or the article [Introduction to Maven Toolchains][8] by Maven maintainer Maarten Mulders).

*Side information: The ballot about the required Java version was hold in March 2024, shortly before Java 22 was released.
One reason Java 17 was chosen over Java 21, because it was (at this time) the second last Java version for which many vendors offer long-term-support.*

### "Fail on severity" parameter
Maven 4 introduces a "fail on severity" build parameter, which will break the build when at least one log message matches the given argument.

The parameter can either be used by its full name (`--fail-on-severity`) or as a short handle (`-fos`).
The parameter is followed by an argument of a log level severity, e.g. `WARN`.

### Optional profiles
Trying to use a nonexistent profile in a build causes the build to fail, as the following command line snippet shows:

```
> mvn compile -Pnonexistent
[ERROR] The requested profiles [nonexistent] could not be activated or deactivated because they do not exist.
```

Maven 4 introduces the possibility to only use profiles when they exist.
To do so the `?` argument was added to the profile parameter.
When using this the build won't break, but an information will be printed twice (at the start and the end).
See the following snippet for an example:

```
> mvn compile -P?nonexistent
[INFO] The requested optional profiles [nonexistent] could not be activated or deactivated because they do not exist.
[...]
[INFO] BUILD SUCCESS
[INFO] ----------------------------------------------------------------------------------------------------------------
[INFO] Total time:  0.746 s
[INFO] Finished at: 2024-12-14T13:24:15+01:00
[INFO] ----------------------------------------------------------------------------------------------------------------
[INFO] The requested optional profiles [nonexistent] could not be activated or deactivated because they do not exist.
```

## Maven plugins and dependencies
As written in the introduction, Maven 4 will contain huge code and API updates (not only because Java 17 language features can be used) and even removals, resulting in breaking very old Maven plugins, which were not updated to the recommended APIs.
For example the "Plexus Containers" dependency was removed - after being deprecated since Maven 3.2 (2010)!

If you are maintaining a Maven plugin, you should test it with Maven 3.9.x and have a close look at the warnings.



# Issue overview
The Maven issue tracker provides a [full list of all resolved issues of Maven 4.0.0][22].
As of 2024-12-14 not all issues are properly linked to the final release and therefore may not be shown in that list.
If you want to see issues resolved in each single (alpha/beta/RC) release, please see the [Maven releases history][10], starting with the alpha versions for Maven 4.0.0.





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
[11]: https://issues.apache.org/jira/browse/MNG-6863
[12]: https://issues.apache.org/jira/browse/MNG-6118
[13]: https://maarten.mulders.it/2020/11/whats-new-in-maven-4/
[14]: https://issues.apache.org/jira/browse/MNG-6754
[15]: https://issues.apache.org/jira/browse/MNG-7038
[16]: https://github.com/apache/maven/pull/1061
[17]: https://issues.apache.org/jira/browse/MNG-624
[18]: https://issues.apache.org/jira/browse/MNG-6656
[19]: https://issues.apache.org/jira/browse/MNG-7051
[20]: https://www.mojohaus.org/flatten-maven-plugin/
[21]: https://blog.soebes.io/posts/2024/03/2024-03-31-maven-4-part-i/
[22]: https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12316922&version=12346477