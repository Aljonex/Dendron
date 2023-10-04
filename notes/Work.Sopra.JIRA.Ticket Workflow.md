---
id: 0jkpyfo53sfvx80ry4vzsp6
title: Ticket Workflow
desc: ''
updated: 1694175241721
created: 1678965369339
---
[Jira Workflow confluence](https://confluence.apak.com/live/display/WIKI/JIRA#JIRA-SXIssues)
## Start
Its fine to use `git push --set-upstream origin feature/<yourFeature>` when setting the upstream to push commits.
Also generally when moving to Sprint Prep generally `None` should be selected for merges, you *must* add an original estimate.

## New ticket process as of 12/05
so the 2 main things are:

test locally and on the environment to free up testers for exploratory and further testing
before merging there must be QA approval

So for example for my SX-65127 (reverse event logic serialisation), once locally tested it should be tested on one of the cloud environments, so SX-65126 is on C74 so the environment is C74, but 65127 is client code I1 so what is the cloud env I'd test it on?

Ah as you've just said you just comment which env you tested it on, does this need to be put in the Devidence document also????

At the moment I think we are largely managing the demos and devidence documents pretty well, but we haven't really taken on the sanity testing of SX tickets on the environment at the end.

 

So what I am proposing is changing the flow to:

- Code Review
    - Demo
        - Discussion between dev and tester around where the changes are in order to build risk based regression cases
    - Devidence
        - Reviewed by tester
- Internal Test
    - Developer tests the SX
        - Key principle that the developer needs to have 100% confidence
            - Can be through UI testing, or Automated testing
            - Minimum of one check in the UI manually
        - Developer adds a comment to the ticket
            - Details of what they tested and if it worked
    - Developer transitions the ticket to External UAT once done
- External UAT
    - Provides the queue of work needing the RBRG tasks to be completed by testers
    - Testers check that all steps have been followed and then close the ticket

 

Hopefully this will remove some of the pressure on the testers to keep up with the head of branch releases, and with the additional exploratory testing time end up increasing the coverage of testing we can do. 

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

### [Merging WMOD ticket](https://confluence.apak.com/live/pages/viewpage.action?pageId=29145650)
1. Model updates -> Model PR up -> Build model PR in jenkins (`WFS Model Feature branch build` in Jenkins, scan multibranch pipeline now, find feature branch) -> Note build version
2. Wfs branch -> insert new model version in POM.xml file -> raise PR, run build to check if works
3. If works -> Merge model PR -> build release version of model -> Note release build version
4. Update model version in wfs branch from the Build in jenkins then merge.

[Model Changes](https://confluence.apak.com/live/pages/viewpage.action?pageId=29145650)