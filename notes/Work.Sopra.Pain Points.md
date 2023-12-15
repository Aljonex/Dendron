---
id: yo8diksbabv7mtibxeuuswc
title: Pain Points
desc: ''
updated: 1701771377388
created: 1679309795074
---

### Populating DB
With modelworld data use the FeatureTestsRunner, debug port is always 8000 with tomcat default

## Restore deleted file in Git
Find the last commit that affected the given path. As the file isn't in the HEAD commit, that previous commit must have deleted it.

`git rev-list -n 1 HEAD -- <file_path>`
Then checkout the version at the commit before, using the caret (^) symbol:

`git checkout <deleting_commit>^ -- <file_path>`
Or in one command, if $file is the file in question.

`git checkout $(git rev-list -n 1 HEAD -- "$file")^ -- "$file"`

### Rebasing
If the branch has moved ahead of where you are and this causes merge conflicts you want to:
- Change to master, in this case `support/8.47/dev` and git pull (can also do `git pull origin support/8.47/dev`)
- Then you want to `git rebase <master-branch>` and resolve the conflicts (there was a comment by lander I could potentially do `git pull --rebase <branch>` from my branch)
- Once rebased a `git status` will return the fact there are commit differences between my branch and the remote and ask for a `git pull` but instead you want to force push `git push -f` to override the remote with what is local

### Local wfs
- Have made it so HoB uses SYSX_USER, 578 uses SYS_USER2, and 388 uses SYSX_USER3 **pain was trying to use SYSX_USER1, which didn't exist**

### Flyway error when running WFS
- Caused by migration change and model world lite runner needs rerunning
- To do this edit the run config of `ModelWorldLiteRunner` and `Modify Options` and add VM args and then input this `-javaagent:C:\Users\alexajones2\ide_resources\libs\aspectjweaver-1.9.5.jar -Xms256m -Xmx4096m -XX:+UseParallelGC -Djava.io.t=pdir=C:\tmp`

### Adding VM args to configurations Intellij
`-javaagent:C:\Users\alexajones2\ide_resources\libs\aspectjweaver-1.9.5.jar -Xms256m -Xmx4096m -XX:+UseParallelGC -Djava.io.tmpdir=C:\tmp`

### setterAspect error
When repopulating a db if this comes up it means you're lacking VM args for the populator, you also need to have `shorten command line` with `JAR manifest - java -cp classpath.jar className [args]` selected.

### Devidence (Acceptance Testing)
https://confluence.apak.com/live/pages/viewpage.action?spaceKey=WIKI&title=Acceptance+Testing

### Day end jobs
There's a recovery tool that allows running of single EOD jobs.


### Restarting cloud env
- In Argo they have been split up via repo, so find the repo first and then you'll see the pods etc
- [Use this flow chart](https://confluence.apak.com/live/pages/viewpage.action?pageId=105243108)

### Upgrade to Java 17
- Download java 17 coretto
- Download newest tomcat 9.0.20
- Download jaspectweaver 1.9.8.jar
- Update tomcat script.properties to use the above 2 (this was issue of running tomcat script)
- Update JUnit template to include
```
-javaagent:c:/Users/alexajones2/ide_resources/libs/aspectjweaver-1.9.8.jar
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
- Update all run config templates to use new javaagent and Java 17
- Update project structure to default to Java 17
    - Make sure modules are using Java 17
    - Update SDK to Java 17

### SX-65619 and SX-65934 (to do with ItemPaymentUtils#negateIfValueReduced)
- Some tests failed due to not being able to call `getFIHolder().getFinancialItem().getEffectOnFunding()`, there are no times when an item won't have a financialItemHolder and therefore financialItem so the data in the tests is wrong, fixing this is the solution, if with a spy then `doReturn().when()` or simply by making a helper method i.e. `InvoiceMergeTest#createChildItemWithEffectOnFunding()`
```java
private IItem createChildItemWithEffectOnFunding(ItemType type, BigDecimal nettCost, BigDecimal vatCost, EffectOnFundingType effectOnFundingType) {
        IItem item = createItem(type, nettCost, vatCost);
        IFinancialItem financialItem = new FinancialItemBuilder()
                .withEffectOnFundingType(effectOnFundingType).build();
        item.setFinancialItemHolder(new FIHolderBuilder().withFinancialItem(financialItem).build());
        return item;
    }

    private IItem createItem(ItemType type, BigDecimal nettCost, BigDecimal vatCost) {
        IItem item = new Item();
        item.setItemType(type);
        item.setNettCost(nettCost);
        item.setVatCost(vatCost);
        item.setItemCost(nettCost.add(vatCost));
        item.setTaxQualifying(true);
        item.setItemCategory(ITEM_CATEGORY);
        return item;
    }
```


### Useful Shortcuts Intellij
- `ctrl+click` - follow method
- `ctrl+f12` - show methods in class


### Test failures
- `DcWizardWorkflowIT` - `testCreateByWizard` failing as "dc is null", not part of the "*quality gate*"


### WFS Multibranch Release
- For stability
    - Find `WFS Release Multibranch`
    - Runs things like jenkinsFile.release on local
        - `RELEASE_VERSION` - auto incremement from jenkins script, for stability it will add `8.47.578.X` - snapshot is always the *next available* version
        - `DEVELOPMENT_VERSION` always one step ahead of release version
        - To check, check the last successful job and click `Parameters` to see how it is setup otherwise
        - Don't stop if its on release or after!!!
        - If the `[maven-jgitflow-plugin]` commit is there don't stop it


### Adding user to cloud env
- Adding myself to dev machines in cloud use this [repo](https://bitbucket.apak.delivery/projects/CONF/repos/asp-dev-machine-conf/browse) and find the `tc` version as that's europe. Add yourself to `users.xml` and sequence gets incremented for password resets.

### Creating Cognito Pool
The template hasn't been updated but this should be added to the bottom of `variables.tf`
```yaml
variable "map_tag_server_id" {
  description = "The value of serverId used for the Amazon Migration Acceleration Program"
  type        = string
}
```

and this should be made to `main.tf`
```yaml
locals {
  name = "<tenant>"
  tags = {
    Service       = "Authentication"
    Role          = "Authentication"
    Owner         = var.owner
    OwnerFunction = var.owner_function
    Customer      = "<customerCode>"
    Stage         = var.stage
    map-migrated  = var.map_tag_server_id
  }
}
```
The tags moved to the top and local variables part. As it has the mapping which is needed now.


### Bitbucket
- Couldn't push to a repo after pulling with `http` and recurse submodules, turns out I just needed to make new http access token with write permission
- Build not available then go to Jenkins [WFS Feature Regression Tests](https://jenkins.apak.delivery/job/WFS%20Feature%20Regression%20Tests/)


### Setup new webapp - Tomcat
Was using Justin's `feature/multiapp` branch as told to in the training (JSF 9) 
```bash
WFS_VERSION=$(resolve_app_version "$APP_HOME")
  
  # Zip local conf into wfs-core
  mkdir -p "$WAR_BASE/configuration"
  PATH_TO_ZIP="configuration/local.conf"
  cp "$WFS_CONF" "$WAR_BASE/$PATH_TO_ZIP"
  cd "$WAR_BASE"
  add_file_to_archive "$WAR_BASE/WEB-INF/lib/wfs-core-$WFS_VERSION.jar" "$PATH_TO_ZIP"
  cd -
```
But it was missing this line in the template for `run_app` which pulls the local-conf into the zipped jar. This was causing major issues when running `388`, so I altered the template so that it would have this as a method.
```bash
#!/bin/bash
source ../util "../script.properties" "../profiles"
source ./run_app.properties

# To use this script, you should set wfs_fleet-specific properties in run_app.properties, and general properties in script.properties.
setup_wfs_deployment() {
  WFS_VERSION=$(resolve_app_version "$APP_HOME")
  
  # Zip local conf into wfs-core
  mkdir -p "$WAR_BASE/configuration"
  PATH_TO_ZIP="configuration/local.conf"
  cp "$WFS_CONF" "$WAR_BASE/$PATH_TO_ZIP"
  cd "$WAR_BASE"
  add_file_to_archive "$WAR_BASE/WEB-INF/lib/wfs-core-$WFS_VERSION.jar" "$PATH_TO_ZIP"
  cd -
}
COMMAND="$1"
ARGS="${@:2}"

case $COMMAND in
  "start")
    setup
    WAR_BASE=$(get_war_base)
    setup_wfs_deployment
    . ../core "$COMMAND" "$WAR_BASE"
    ;;
  *)
    . ../core "$COMMAND" "$ARGS"
    ;;	
esac
```
My example setup for 388 is as follows:
```bash
$ ./setup_new_webapp

Name of app in snake case [e.g. test_app]: wfs_388

Attempting to create directory wfs_388 in current directory...

Directory does not exist. Creating directory wfs_388...

Directory wfs_388 created in current directory.

Creating scripts in wfs_388 directory...

Scripts for wfs_388 created.

Creating .properties files in wfs_388 directory ...
Persisting property APP_NAME=wfs_388 to wfs_388/run_wfs_388.properties...

Path to your app's repo [e.g. C:\Users\user\git\WFS]: C:\Users\alexajones2\git\wfs-388
Persisting property APP_HOME=C:\Users\alexajones2\git\wfs-388 to wfs_388/run_wfs_388.properties...

Folder name of module to be deployed [e.g. wfs-ui, or blank if not a multi-module maven project]: wfs-ui
Persisting property DEPLOYMENT_MODULE=wfs-ui to wfs_388/run_wfs_388.properties...

Fully qualified path to exploded war (leave empty if generated by maven)
[e.g. C:\Users\user\git\wfs\wfs-ui\target\wfs-ui-8.70-SNAPSHOT]:
Persisting property WAR_PATH= to wfs_388/run_wfs_388.properties...

Path to web application content root (directory where .xhtml files are located, relative to application root): WebRoot
Persisting property WEBROOT=WebRoot to wfs_388/run_wfs_388.properties...

Path to JDK (leave empty if using your default JAVA_HOME environment variable): C:\Program Files\Amazon Corretto\jdk11.0.21_9
Persisting property JDK_PATH=C:\Program Files\Amazon Corretto\jdk11.0.21_9 to wfs_388/run_wfs_388.properties...

Path to Maven (leave empty if using your default M2_HOME environment variable)
Persisting property MAVEN_HOME= to wfs_388/run_wfs_388.properties...

HTTP port to access application [defaults to 8080 if left blank]: 8091
Persisting property HTTP_PORT=8091 to wfs_388/run_wfs_388.properties...

The debug port exposed to connect a debugger to the running web application [defaults to 8000 if left blank]:

Context path for URL (i.e. http:://localhost:8091/<context path>) [defaults to WFS_388 if left blank]:
Persisting property CONTEXT_PATH=WFS_388 to wfs_388/run_wfs_388.properties...

Setting Catalina base directory to $HOME/servers/$CONTEXT_PATH-tomcat...
Persisting property CATALINA_BASE_DIR=$HOME/servers/$CONTEXT_PATH-tomcat to wfs_388/run_wfs_388.properties...
Resolving operating system...Persisting property OS=WINDOWS to wfs_388/run_wfs_388.properties...
Setup completed

If you wish to amend any of the properties, you may manually edit the run_wfs_388.properties file under the newly created wfs_388 directory.
You may also want to refer to script.properties at the root of this repo to edit properties that can be applied to all apps running on tomcat via these scripts.
Properties in script.properties which are also defined in run_wfs_388.properties will be overwritten, so you can manually define those variables in run_wfs_388.properties in case you missed any during the setup.
Lastly, you may want to review run_wfs_388.properties and script.properties to check if you have incorrectly entered any of the properties.

To deploy your web app, navigate to the new wfs_388 directory under scripts in your bash and try running ./run_wfs_388
```


### Solving problems
- Check logs on relevant machine for time scale stated and look through logs
- Recreate locally
- If unable to, try and mirror important data points, check differences in business rules of products (relative to logs and problem scope)
- Then retry recreating locally

### For Argo
- EU-west is just argo.apak.delivery
- For other envs, `argo-<env (i.e. ap-southeast-4).apak.delivery`