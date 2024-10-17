---
id: gcyr1rohomuve0xylw3fgyv
title: SX-69087-DGSCP
desc: ''
updated: 1724843902723
created: 1724839409669
---
### Finishing this ticket
Rich had done all the code changes for this and basically I was tasked with completing AC, testing (unit and Integration) and then booking a demo.

### Issues
First hiccup came up when despite having selected it in both transactional and non-transactional day end jobs, selecting it in the UI reports (shouldn't have to but there's a bug where you have to) and having it selected it in the monthly dealer group reports it wouldn't show up.

Rich got me to go to PFC level
- Run recovery tool, end of the month just run (in this case, last day of may)
- Select report generation and execute

Rich got me to debug
- `ReportGeneration` - line in the if statement checking its month end, then running the `buildTasksFromConfigLists` method that gets called in evaluate expression. It was present.
- Then checked the `OrganisationGroupReportGenerator`, debugged runReport (it was there)
- Checked `#getReportsForChildOrganisations` and debugged the for loop, it was `topLevel` as expected but the `isGlobalDealer(childOrgId)` was false, meaning there was no Dealer set as head office user on the dealer group
- Had to go into dealer 3, or dealer 1, and set the other as their parent org.

Still wasn't showing up on Dealer level, through lots of debugging of the generatedReportsController (for the page) and following lots of spooling etc.
- Came to realisation that the default visibility is DEALING_COMPANY
- Had to add an overridden method to `reportLayout` of `getCompletedReportVisibleTo()` and return `DEALING_COMPANY` and `DEALER`.