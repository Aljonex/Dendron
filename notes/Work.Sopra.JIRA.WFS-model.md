---
id: bg83pzso5furhv19gh6rk21
title: WFS-model
desc: ''
updated: 1682428760400
created: 1682414261141
---
## Remove HQL from model SX-65093
Firstly had to git clone a new repo [see here](https://bitbucket.apak.delivery/projects/BRK/repos/wfs-model/browse?at=refs%2Fheads%2Fsupport%2F55.0.2%2Fdev)

Chris informed me it was a slightly more challenging ticket than I'd done so far, and its a [flyway](https://flywaydb.org/) system that migrates the schema of the 3 different databases: *oracle, postgres, and hsql*. But HSQL isn't being used anymore so those steps can be removed, look through the model project and delete everything that tries to use HSQL.

#### FLYWAY
Extends DevOps to your databases to accelerate software delivery and ensure quality code. 
It builds on application delivery processes to automate database deployments.