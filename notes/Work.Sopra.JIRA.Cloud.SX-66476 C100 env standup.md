---
id: w44tzypf04q9sja42jmw1de
title: SX-66476 C100 env standup
desc: ''
updated: 1698403884424
created: 1698240200293
---
For the jira ticket [SX-66476](https://jira.apak.com/browse/SX-66476) regarding standing up a new cloud env with HoB 8.47.

### Steps
1. [Cloud Env Page confluence](https://confluence.apak.com/live/display/WIKI/Cloud+Environments) and read up on the `Internal Test Environment setup page`
2. Pulled the `cloud-deploy-template` to make sure I was up to date
3. Just in case, re-ran `pip install -r requirements.txt`
4. When using *GitBash* can't use the `python` command, but can use `py main.pyw` and it'll do the same, then followed with some instructions from James linked to the ticket which were much the same as [Standard Application and Resource Definition](https://confluence.apak.com/live/display/WIKI/Standard+Application+and+Resource+Definition)
5. Adjusted the values in the program to match the ticket i.e. Max's email and things of this nature
6. Some points from James
    - One of the key ideas behind tenancy is that data should never leave a tenant - that looks much better, you'll need to adjust the sfpw-1 in environmentidentifier and subdomain to be "hob"
    - The identifier of the machine - composed of Service Namespace + "-" + environment identifier
    - Subdomain - the URL of the machine so c100-test-hob would be https://c100-test-hob.apak.delivery/


Made a mistake, so `git revert <commitIdOfMistake>` then merge to master, sync the resources in Argo with **prune** ticked.