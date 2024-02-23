---
id: h9amlurfv1sxgnq2l4y0lz2
title: Oracle-18c
desc: ''
updated: 1707403935423
created: 1707403798033
---
I set this up using local dev compose and then downloading oracle SQL developer and changing local.conf
```
migration.flyway {
          locations = "db/migration/oracle, db/migration/common"
          nonPartitionLocations = "db/common_data/migration/oracle, db/common_data/migration/common"
#           locations = "db/migration/postgresql, db/migration/common"
#           nonPartitionLocations = "db/common_data/migration/postgresql, db/common_data/migration/common"
          createcommonhibernatesequence = false
          repair = true
          purge = true
          #auto = false
        }
```
Had to sudo the `./pull_image.sh` in `scripts/ecr` but this was because of docker remnants from docker desktop.