---
id: 0vvurq25uxm838qtkufz8s9
title: Maven
desc: ''
updated: 1669112568103
created: 1668424982229
---
## Build Lifecycle
[[Maven Build Lifecycle |https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html]] means the process of building and distributing a particular artifact (project) is clearly defined. 
You only need a small set of commands to build any Maven project and the **POM** will ensure that you get the desired results.

The build phases:
- `validate` - validate the project is correct and all necessary information is available
- `compile` - compile the source code of the project
- `test` test the compiled source code using a suitable unit testing framework. These tests shouldn't require the code be packaged or deployed.
- `package` - take compiled code and package in its distributable format, like JAR.
- `verify` - run any checks on results of integration tests to ensure quality criteria are met.
- `install` - install package into local repo, for use as dependency in other projects locally.
- `deploy` - done in build environment, copies final package to remote repo for sharing with other developers and projects.

To call the stage you only need to call the last build phase to be executed with `mvn {phaseName}` and it will call all default lifecycle phases before that in order.

### POM
[[POM Documentation | https://maven.apache.org/guides/introduction/introduction-to-the-pom.html]] <br>
[[The POM Reference | https://maven.apache.org/pom.html#Introduction]] <br>
**Project Object Model** is the fundamental unit of work in Maven.
Its an XML file that contains project and configuration details information, which in turn is used by Maven to build the project.
> Examples:
- Build directory: `target`
- Source directory: `src/main/java`
- Test directory: `src/test/java`

The Super POM is the default POM, and all POMs extend this unless explicitly set

### Tags
- **Scope** - used to limit transitivity of a dependency, also affects classpath used for various build tasks: [[ see more | https://stackoverflow.com/questions/26975818/what-is-scope-under-dependency-in-pom-xml-for]] <br>
[[Maven Documentation on Scope Dependency | https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#Dependency_Scope]]

#### Minimal POM
Minimum requirements are:
- `project` root
- `modelVersion` - should be set to 4.0.0
- `groupId` - id of projects group
- `artifactId` - id of the artifact (project)
- `version` - the version of the artifact under the specified group
![](/assets/images/2022-11-14-11-29-57.png)

### Surefire Plugim
Automatically includes all test classes with following patterns:
- `**/Test*.java` = all subdirectories and all Java filenames that start with "*Test*"
- `**/*Test.java` = all subdirs and all Java files that end with "*Test*"
- `**/*Tests.java` = as above but ends in "*Tests*"
- `**/*TestCase.java` = ends in "*TestCase*"

When a test has been ignored in the exclude tag
![](/assets/images/2022-11-22-10-21-35.png) 
It should get through on the `mvn <command>` for example here I used `mvn verify` as it is excluded in the pom file FOR MAVEN.
However, JUnit tests will still return it as a failing test:
![](/assets/images/2022-11-22-10-22-42.png)
