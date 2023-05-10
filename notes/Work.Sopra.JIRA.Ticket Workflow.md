---
id: 0jkpyfo53sfvx80ry4vzsp6
title: Ticket Workflow
desc: ''
updated: 1682679603643
created: 1678965369339
---
[Jira Workflow confluence](https://confluence.apak.com/live/display/WIKI/JIRA#JIRA-SXIssues)
## Start
Its fine to use `git push --set-upstream origin feature/<yourFeature>` when setting the upstream to push commits.
Also generally when moving to Sprint Prep generally `None` should be selected for merges, you *must* add an original estimate.

## 8.47 
### When closing
**Commit** - every version where code relating to the ticket has been committed, check this on [line 333](https://bitbucket.apak.delivery/projects/WFS/repos/wfs/browse/pom.xml?at=refs%2Fheads%2Fsupport%2F8.47%2Fdev)

**Fix Version** - version where ticket was fixed/merged, should always be 2 (the branch and a specific version which is the latest commit), to find the version go to `8.47/dev` and go to the wfs-core and find the *SNAPSHOT* version from the POM.

**Fix Strategy and Risk Assessment** - If this was to break something in production then how would it be fixed? In Risk Assessment outline the risks and what they would cause and then how to monitor that (i.e. added logging to monitor the job)

Then rename the title of PR and remove feature and have ticket in `CAPITALS-<TICKETNUM>`

### Before pushing
`Pull latest, rebase onto it, then push`

**MERGE** never means merge, for example "I'd like you to complete the merges to 8.47.220 and 8.47.388...for X changes" most likely means cherry pick where the fixes occurred onto the branch.


### After pushing
Go to bitbucket and click the three dots and build and select `Regression Tests` parameters

### Risk Assessments
[James Reynolds](https://confluence.apak.com/live/pages/viewpage.action?spaceKey=~james.reynolds&title=Risk+Analysis+8.47+Draft)

### Ticket Types
CMN - for APAK common work
WMOD - for WFS model work
SX - for WFS work
ETPS - for production and branch work

### Create WMOD ticket
Edit the ETPS ticket issue to see everything needed, click create:
1. Project - WMOD
2. Issue Type - Task
3. Details just add in all the details from ETPS ticket
4. Copy Summary and Description etc
5. Client Code I1
6. No sprint or epic link
7. Link the ticket with SX ticket made and ETPS ticket

### Merging WMOD ticket
1. Model updates -> Model PR up -> Build model Pr in jenkins -> Note build version
2. Wfs branch -> insert new model version in POM.xml file -> raise PR, run build to check if works
3. If works -> Merge model PR -> build release version of model -> Note release build version
4. Update model version in wfs branch from the Build in jenkins then merge.