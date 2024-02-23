---
id: y5v95lwxge761y2z1dlkp57
title: Cloud
desc: 'All things wonderful and cloudy'
updated: 1704369257913
created: 1685959110858
---
So with cloud you need to pull both the `wfs-production` and `sandbox` repos from the relevant AWS region, for this case it was EU-WEST-1.

For [deleting cloud environments](https://confluence.apak.com/live/display/WIKI/Deleting+a+Cloud+Environment), follow stage 2, then push this as `feature/<etps/hbrd>-<ref>-<typeOfUpdate>-<prod/sandbox>-<envNum>-<words>`, once this is MERGED, then DELETE THE FILES but until then do NOT DO THIS.

### Logs
Anything cloud-based is on *Kibana*, so *PROD* (production/live), and anything hosted on the cloud (generally not *dev* (developer) as these are **mostly** done locally, or *acc* (acceptance testing)). 
- *batest* environments are for internally testing new changes