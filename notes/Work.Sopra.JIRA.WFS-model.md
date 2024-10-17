---
id: bg83pzso5furhv19gh6rk21
title: WFS-model
desc: ''
updated: 1724153364866
created: 1682414261141
---
## Remove HQL from model SX-65093
Firstly had to git clone a new repo [see here](https://bitbucket.apak.delivery/projects/BRK/repos/wfs-model/browse?at=refs%2Fheads%2Fsupport%2F55.0.2%2Fdev)

Chris informed me it was a slightly more challenging ticket than I'd done so far, and its a [flyway](https://flywaydb.org/) system that migrates the schema of the 3 different databases: *oracle, postgres, and hsql*. But HSQL isn't being used anymore so those steps can be removed, look through the model project and delete everything that tries to use HSQL.

Usually follow the below steps but for this as a test fails you just merge it when Chris approves, then go into `Jenkins -> WFS Model Release Multibranch -> support/55.0.2/dev`, then `Build with Parameters (remove dry run)`, once this has built check the logs for the model version i.e. `55.0.2.164` and make a branch from the SX ticket with the `wfs-model.version` changed to this in the pom, and create a PR.

[How to make model changes](https://confluence.apak.com/live/pages/viewpage.action?pageId=29145650)

When building just on `WFS Model Release Multibranch -> <branch>` I could also get the new tag from
```
2023-06-29 10:00:38.6#37   - [deleted]         support/55.0.2/release/55.0.2.163
2023-06-29 10:00:38.637   * [new tag]         55.0.2.163 -> 55.0.2.163
```


### [Merging WMOD ticket](https://confluence.apak.com/live/pages/viewpage.action?pageId=29145650)
1. Model updates -> Model PR up -> Build model Pr in jenkins -> Note build version
2. Wfs branch -> insert new model version in POM.xml file -> raise PR, run build to check if works
3. If works -> Merge model PR -> build release version of model -> Note release build version
4. Update model version in wfs branch from the Build in jenkins then merge.


#### FLYWAY
Extends DevOps to your databases to accelerate software delivery and ensure quality code. 
It builds on application delivery processes to automate database deployments.

- Java files can be modified as have no associated checksum
- If changing an sql file you need to add a recovery step
- With new breakout of model now you have to be aware that `flyway.conf` and `flyway.xml` have changed, look at `PartitionAwareFlyway` to see running order but basically the steps run as `Recovery -> Main -> Data Update -> Schema Deletion` and to be able to delete a step and have it readded you need to set `Schema Deletion.disabled` to true in flyway.conf as such:
```conf
wfs {
  flyway {
	migration.flyway {
	;   locations = "db/migration/oracle, db/migration/common"
	;   nonPartitionLocations = "db/common_data/migration/oracle, db/common_data/migration/common"
	;   auto = true
	;   repair = false
	;   purge = false
	;   createhibernatesequences = false
	;   createcommonhibernatesequence = true
	;   outoforder = true
    ;   dataUpdate {
    ;     auto = true
    ;     locations = "flyway/data/oracle, com/apakgroup/flyway/data/common"
    ;   }
      schemaDeletion {
        disabled=true
        ; auto = false
        ; locations = "flyway/schemaDeletion/oracle"
      }
    ;   recovery {
    ;     disabled = false
    ;     auto = true
    ;     locations = "flyway/recovery/oracle"
    ;   }
    ; }
	; internalaccountcreation {
	;   initialautocreatecreditline = false
	; }
  }
}
```
As this links to here in PartitionAwareFlyway:
```java
public void autoMigrate() throws Exception {
         if(!(repairOn || purgeOn || autoMigrateOn) || hardDisabled){
            LOGGER.info("Skipping automigration for {} ({}, {}, {}, {})", beanName, repairOn, purgeOn, autoMigrateOn,
                    hardDisabled);
            return;
        }
        LOGGER.info("Auto Migration Triggered for partitions {}", partitionConfiguration.partitionSet());
        String oldPartition = PartitionContextHolder.getPartition();
        for (String partition : partitionConfiguration.partitionSet()) {
            List<Callback> actualCallbacks = new ArrayList<>(callbacks);
            if (purgeOn) {
                if (!repairOn) {
                    LOGGER.warn("Purge enabled but repair disabled - this results in no actual invocations");
                }
                actualCallbacks.add(new FlywayPurgeCallback(locations));
            }
            if (autoMigrateOn) {
                actualCallbacks.add(new HibernateSequenceCallback(rdbms, dialect, partitionConfiguration));
            }

            Flyway flyway = createFlyway(partition, actualCallbacks);
            if (repairOn) {
                LOGGER.info("Triggering Flyway Repair for partition {}", partition);
                flyway.repair();
            }
            if (autoMigrateOn) {
                LOGGER.info("Triggering Flyway Migration for partition {}", partition);
                flyway.migrate();
            }
        }
```