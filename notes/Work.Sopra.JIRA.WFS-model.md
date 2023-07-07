---
id: bg83pzso5furhv19gh6rk21
title: WFS-model
desc: ''
updated: 1688468999197
created: 1682414261141
---
## Remove HQL from model SX-65093
Firstly had to git clone a new repo [see here](https://bitbucket.apak.delivery/projects/BRK/repos/wfs-model/browse?at=refs%2Fheads%2Fsupport%2F55.0.2%2Fdev)

Chris informed me it was a slightly more challenging ticket than I'd done so far, and its a [flyway](https://flywaydb.org/) system that migrates the schema of the 3 different databases: *oracle, postgres, and hsql*. But HSQL isn't being used anymore so those steps can be removed, look through the model project and delete everything that tries to use HSQL.

Usually follow the below steps but for this as a test fails you just merge it when Chris approves, then go into `Jenkins -> WFS Model Release Multibranch -> support/55.0.2/dev`, then `Build with Parameters (remove dry run)`, once this has built check the logs for the model version i.e. `55.0.2.164` and make a branch from the SX ticket with the `wfs-model.version` changed to this in the pom, and create a PR.

[How to make model changes](https://confluence.apak.com/live/pages/viewpage.action?pageId=29145650)

When building just on `WFS Model Release Multibranch -> <branch>` I could also get the new tag from
```
2023-06-29 10:00:38.637   - [deleted]         support/55.0.2/release/55.0.2.163
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