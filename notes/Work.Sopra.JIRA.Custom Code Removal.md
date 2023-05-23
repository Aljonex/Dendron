---
id: paotjaa7a6v9oy0tndgx2wn
title: Custom Code Removal
desc: ''
updated: 1684418366448
created: 1681810768249
---
### SX-64970 C21
- Had to remove `c21DouglasFeedFile` from `BaseSystemX.xml` and then update where this was called in the same file with the default, also remove `DouglasGuardianFeedIT.java` as it was only for C21.
- There were also removals of references to C21 in files like `WfsEodProcessType.java` and `CategoryRunner.java` and `custom.conf`
- The `DouglasGuardianFeed.java` file was only pertinent to C21, and had to remove referenced bean in config so that I could reload the webapp
- Had a flyway error occur which just meant the schema fell out of sync so had to repopulate the db with modelworld - for lite version use `FeatureTestRunner` for full use `ApakModelWorldRunner.initialiseDb()` to get the dump then use `ApakModelWorldDMPRunner`