---
id: t0v01hivgmbaa9ed9ublak0
title: Configurations
desc: ''
updated: 1688029971952
created: 1688029137323
---
# Tomcat
## Login to wfs
User: `ADMINPFC`
<br>Password: `@p@kv6`

# Intellij
## Project Structure
For anything after 8.47.480 -> `Java 17`

## Intellij Run Configuration Templates

### maven
(for wfs base)
<br>Run
<br>`clean package -pl wfs-ui -am -DskipTests -T1C`

Working directory
<br>`wfs-parent`

Profiles
<br>`skip-tests intellij`

JRE: 17
<br> VM Options
<br>`-javaagent:C:\Users\alexajones2\ide_resources\libs\aspectjweaver-1.9.8.jar
-Xms256m
-Xmx4096m
-XX:+UseParallelGC
-Djava.io.tmpdir=C:\tmp`

### remote jvm debug
defaults but port `8000` (tomcat)

### JUnit
VM options
```
-javaagent:C:/Users/alexajones2/ide_resources/libs/aspectjweaver-1.9.8.jar
-Xms256m
-Xmx4096m
-XX:+UseParallelGC
-Djava.io.tmpdir=C:\tmp
--add-exports=java.base/jdk.internal.ref=ALL-UNNAMED
--add-exports=jdk.unsupported/sun.misc=ALL-UNNAMED
--add-opens=java.base/java.io=ALL-UNNAMED
--add-opens=java.base/java.lang.reflect=ALL-UNNAMED
--add-opens=java.base/java.lang=ALL-UNNAMED
--add-opens=java.base/java.math=ALL-UNNAMED
--add-opens=java.base/java.nio=ALL-UNNAMED
--add-opens=java.base/java.util.concurrent=ALL-UNNAMED
--add-opens=java.base/java.util=ALL-UNNAMED
--add-opens=java.base/javax.xml=ALL-UNNAMED
--add-opens=java.base/sun.nio.ch=ALL-UNNAMED
--add-opens=java.management/sun.management=ALL-UNNAMED
--add-opens=java.naming/com.sun.jndi.ldap=ALL-UNNAMED
--add-opens=jdk.management/com.ibm.lang.management.internal=ALL-UNNAMED
--add-opens=jdk.management/com.sun.management.internal=ALL-UNNAMED
```

Type of resource to search for test: `all in directory`
<br> Then choose directory i.e. `C:\Users\alexajones2\git\wfs\wfs-core\test`
<br>Shorten command line: `JAR manifest`