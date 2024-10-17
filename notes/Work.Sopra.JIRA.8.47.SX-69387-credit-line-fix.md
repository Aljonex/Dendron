---
id: 7oni5drx807tm4ag2lnu08x
title: SX-69387-credit-line-widget-fix
desc: ''
updated: 1726647468198
created: 1726646868986
---
## Disclaimer
I didn't realise when picking up this ticket that this had already been fixed on [SX-68347](https://jira.apak.com/browse/SX-68347) for the HoB and so was doing a lot of debugging to solve it.

## Points of note
- PortalCreditLimitController
- PortalService
- PortalRefreshJob
- PortalDelegatingUIController

### Breakpoints
Started with breakpoints on `PortalService#populatePortalObjects` and eventually `PortalService#getCreditLimieRowsByOwnerFromCache`, also `PortalRefreshJob#executeForPartition` and in `PortalCreditLimitController#creditLines #getCreditLines #getCreditLinesModel` and it turned out there was excess logic in `getCreditLinesModel` which was hiding the information needed. Weird that it was executing this logic in the controller instead of the service layer.