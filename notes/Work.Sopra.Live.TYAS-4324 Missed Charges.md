---
id: 1g5qxiorc5mcnw5vkgi5mam
title: TYAS-4324 Missed Charges
desc: ''
updated: 1722522476914
created: 1722521941708
---
Given the xlsx file and everything available, started solving in various ways but effectively saw that on the loans:
- Searching in top right by the given VRM
- Clicking all transaction history
- Clicking the date that was the problem (31/05/2024)
- Could see there were no transaction links
- Also when viewing system item search (which should generate about monthly, as is rolled up collection of charges on loans) - there was one at the start of every month except June leading me to know that that's why May's charges weren't collected
- Also looking at item **FG72XDD** I could see the difference and lack of *transaction links* table population on the 31/05/2024 entry when compared with 30/06/2024

Based on this I knew we were missing the System Items and despite the loans having these interest at the end of May with `Status: ROLLED` they weren't being collected because of the lack of System Item generated. 
**System Items** are from the `item` table in the database with type of `CHARGES` and are a way of collecting all charges across a dealers loan items so it can be one lump payout.

Chris had said that either:
- These non-collected items are in system item and **rolled but not paid**
- Or they're not present as system items but the status was still changed to rolled

It was the latter, in which case there needs to be a shift of their due date I believe and a change of status to PENDING from ROLLED so that on the next EOM they get collected truly.

So to find the items as written in the xlsx file, used SQL runner on the `cs-tc3-dev047.apak.delivery/WFS/portal.faces` env and used this query:
```SQL
SELECT *
FROM mergetransaction
WHERE MergeType = 'ROLLED'
	AND creationdate = TO_DATE('2024-05-31', 'YYYY-MM-DD')
	AND (
		MergeKey LIKE '006720%'
		OR MergeKey LIKE '006668%'
		OR MergeKey LIKE '000487%'
		OR MergeKey LIKE '006954%'
		OR MergeKey LIKE '660066%'
		)
```
And this returned 8 results, on the xlsx there are 4 sections each with multiple loans missing and their values and the total missing per section, these 8 values when collated by organisation id (`mergeKey`, also part of `paymentnarrative`) they added to the non-collected values.

This apparently occurred due to payment sources on DEALERS not having advanced settings of payment source set to all.