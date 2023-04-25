---
id: tt20j8reqb1pkv1qyhuwnc7
title: Adding Jobs to DC
desc: ''
updated: 1680787720469
created: 1680708879041
---
### SX-64930 Add ProducedFilesCleanup and HealthCheck as default on DC creation

In the end after much deliberation the solution was relatively simple, the ticket lacked some information so:
- First port of call was to search in files for `HealthCheck` and `ProducedFilesCleanup` using `shift+shift` and `ctrl+shift+f` but neither of these bore fruit
- The next port of call was to run the local WFS and then use the remote debugger (attached to port 8000 as is tomcat standard) and then follow the easiest DC creation in `Admin-> Organisation -> Dealing Companies -> Create with wizard`, from here upon saving there was a file being called called `wizardConfiguration.json` and there were sections per region for both `TransactionalDayEnds` and `NonTransactionalDayEnds` which meant all I had to do was add `producedFilesCleanup` and `healthChecks` to NTEOD sections