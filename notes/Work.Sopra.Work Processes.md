---
id: g70kze1dz3xlaa7246zjdw8
title: Work Processes
desc: ''
updated: 1686141102111
created: 1678976821883
---
## Holiday
Have you:
- Logged in Record Activity (F2F)
- Logged in [Jira dashboard](https://jira.apak.com/secure/Dashboard.jspa?selectPageId=16047)
- Set out of office status on `File -> Automatic Replies` in outlook
- Set away status in Teams

### 8.47
Hey All.  I want to change our dev testing process a bit on 8.47 to be more in line with the established embedded testing process in order to bring more of the testing effort of tickets in to the development side in order to free you all up to get more of the risk based regression/exploratory testing kind of things done.

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