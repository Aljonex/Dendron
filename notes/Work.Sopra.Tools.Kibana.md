---
id: 9nty61cu5zzyixivftr6hn5
title: Kibana
desc: ''
updated: 1695899552514
created: 1695293845464
---
*Kibana* - useful system for looking at logs of all systems, live, dev environments and otherwise, can look specifically for environment and maybe my username, change times etc

### Kibana logs for SX-65126
Before adding logging there were only 3 logs being shown, when the LOAN_MOVEMENT_SUMMARY_REPORT started, then saying something to do with `LOANMVMT` and then the report finishing, so logs were added to commonly seen methods throughout the generation in the `System Admin -> Recovery Tools -> Thread Dump Recovery Tool` by repeatedly loading this and copying the contents across to a notepad file to see what was being called throughout.

Ideal is to add the highest level possible, so for example do DailyReportScriptedDataSetPage1, 2, and 3, start with that and drill into service and just say do the service layer stuff, then once you know where the time has come from drill into the DAO.


### Kibana presentation for Noida devs
General flow of presentation:
- Intro to Kibana
    - Use of elastic search with it
    - Why we use it
    - Link for access
    - How to start looking through environments - all live environments, prod and non prod but not dev
- Where the setups of cloud envs are set so finding hostname is easier https://bitbucket.apak.delivery/projects/EUW1NP
- Key points of note on the discover dashboard + cheat sheet and KQL/elastic syntax
- Demo of example finding some key details, maybe even searching when error occurs
- Cause some problem on sandbox environment (like validation error) and then open breakout rooms and ask them how they'd find the problem, potentially do it live and ask them to find a solution and come back and say the key things they used
- Final tips (heavy use of filters, saving link saves search results etc)
