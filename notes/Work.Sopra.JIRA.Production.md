---
id: rij31i4u7eqhufiknvt7u21
title: Production
desc: ''
updated: 1682428769745
created: 1682083715896
---
### ETPS-2024 delete c101 production files
- First update `wfs-prod-<number>` to allow deletion (true and removing deletion protection) and in the schema `protection: unlocked` also don't forget to set the `deletion_protection.enabled` flag on the ingress to `false`.