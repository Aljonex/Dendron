---
id: bx7rsee20jhrumxmjh949ed
title: SX-67304 Internal Account Dealer Statement Report
desc: ''
updated: 1710473174809
created: 1709295799531
---



## Errors
### Report Layout
` Group data columns cannot be defined both in data set and in rendition level(s)` - FIX: 
```java
newDataSet(reportFormat).scriptedDataSet(internalAccountDealerStatementScriptedDataSet)
    .fields(FIELDS)
    .rendition(
            newRendition(reportFormat, PDF, EXCEL)
            .fontSize(PDF, 7)
            .titleLabel("transaction_summary")
            //.empty(getTransactionDetailsSummaryHeader(rp, true).style(ReportGenStyleType.LEVEL1_HEADER))
            .level(newLevel()
            //.dataColumns(FIELDS)
                .level(
                    newLevel()
                    //.groupColumns(LOAN_ID)
```
Re-added fields at top level and commented out grouping columns in a nested level.

### Tests
- `DAOIT` - problem when testing with dates, didn't make use of the **DateUtil** and instead tried to use Java `Date.util` which returns years in Gregorian calendar years which is ours +1900, not useful.