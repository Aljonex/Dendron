---
id: rywxnm95mbuyyzty7bmh4kz
title: Cloud
desc: ''
updated: 1678901501999
created: 1678900208720
---
So with cloud you need to pull both the `wfs-production` and `sandbox` repos from the relevant AWS region, for this case it was EU-WEST-1.

For [deleting cloud environments](https://confluence.apak.com/live/display/WIKI/Deleting+a+Cloud+Environment), follow stage 2, then push this as `feature/<etps/hbrd>-<ref>-<typeOfUpdate>-<prod/sandbox>-<envNum>-<words>`, once this is MERGED, then DELETE THE FILES but until then do NOT DO THIS.