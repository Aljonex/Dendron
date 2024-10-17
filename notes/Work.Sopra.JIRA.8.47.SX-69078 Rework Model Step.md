---
id: z4yz8mvl4wmmf7xyzzsjmoc
title: SX-69078 Rework Model Step
desc: ''
updated: 1725012644539
created: 1725009849388
---
### Solution
Basically the route to solution of this was managing to debug the `V26_15_0_18_3__PortRollupJobs.java` in the `updateFinancialItems` method, there was a switch case assigning something but then there was an `updateClause` where regardless, provided the financial item IDs were found there would have `WITHIN_ROLL_UP` applied, meaning that timing type would overwrite null, empty, or a previously held value. We wanted previously held values to stay as they were.
<br>**OLD CODE**
```Java
switch (tc) {
                case COLLECTED_NETT:
                    financialItemRollupConfigIdPath = financialItemPath.get(COLLECTED_NETT_ROLLUP_CONF_ID, Long.class);
                    timingTypePath = financialItemPath.getString("COLLECTEDNETTTIMING");
                    break;
                case COLLECTED_TAX:
                    financialItemRollupConfigIdPath = financialItemPath.get(COLLECTED_TAX_ROLLUP_CONF_ID, Long.class);
                    timingTypePath = financialItemPath.getString("COLLECTEDTAXTIMING");
                    break;
                case PAID_OUT_NETT:
                    financialItemRollupConfigIdPath = financialItemPath.get(PAID_NETT_ROLLUP_CONF_ID, Long.class);
                    timingTypePath = financialItemPath.getString("PAIDOUTNETTTIMING");
                    break;
                case PAID_OUT_TAX:
                    financialItemRollupConfigIdPath = financialItemPath.get(PAID_TAX_ROLLUP_CONF_ID, Long.class);
                    timingTypePath = financialItemPath.getString("PAIDOUTTAXTIMING");
                    break;
                default:
                    throw new IllegalArgumentException("Null TimingCombination enum value passed");
            }

            for (List<Long> fiIdsBatch : Iterables.partition(fiIds, 1000)) {
                SQLUpdateClause updateClause = queryFactory.update(financialItemRelativePath);
                updateClause.set(financialItemRollupConfigIdPath, rcId)
                        .set(timingTypePath, "WITHIN_ROLL_UP")
                        .where(financialItemPath.get("ID", Long.class).in(fiIdsBatch))
                        .execute();
```
<br>**NEW CODE**
```Java
String timingColumn;
            switch (tc) {
                case COLLECTED_NETT:
                    financialItemRollupConfigIdPath = financialItemPath.get(COLLECTED_NETT_ROLLUP_CONF_ID, Long.class);
                    timingTypePath = financialItemPath.getString(COLLECTED_NETT_TIMING);
                    timingColumn = COLLECTED_NETT_TIMING;
                    break;
                case COLLECTED_TAX:
                    financialItemRollupConfigIdPath = financialItemPath.get(COLLECTED_TAX_ROLLUP_CONF_ID, Long.class);
                    timingTypePath = financialItemPath.getString(COLLECTED_TAX_TIMING);
                    timingColumn = COLLECTED_TAX_TIMING;
                    break;
                case PAID_OUT_NETT:
                    financialItemRollupConfigIdPath = financialItemPath.get(PAID_NETT_ROLLUP_CONF_ID, Long.class);
                    timingTypePath = financialItemPath.getString(PAID_OUT_NETT_TIMING);
                    timingColumn = PAID_OUT_NETT_TIMING;
                    break;
                case PAID_OUT_TAX:
                    financialItemRollupConfigIdPath = financialItemPath.get(PAID_TAX_ROLLUP_CONF_ID, Long.class);
                    timingTypePath = financialItemPath.getString(PAID_OUT_TAX_TIMING);
                    timingColumn = PAID_OUT_TAX_TIMING;
                    break;
                default:
                    throw new IllegalArgumentException("Null TimingCombination enum value passed");
            }

            for (List<Long> fiIdsBatch : Iterables.partition(fiIds, 1000)) {
                SQLUpdateClause updateClause = queryFactory.update(financialItemRelativePath);

                updateClause.set(financialItemRollupConfigIdPath, rcId)
                        .where(financialItemPath.get("ID", Long.class).in(fiIdsBatch))
                        .execute();
                updateClause.set(timingTypePath, "WITHIN_ROLL_UP")
                        .where(financialItemPath.get(timingColumn, String.class).isNull())
                        .where(financialItemPath.get("ID", Long.class).in(fiIdsBatch))
                        .execute();
            }
```

So effectively I added a check for if the timing column was null, and then it would add `WITHIN_ROLL_UP`.

### Useful links
- [James original PR](https://bitbucket.apak.delivery/projects/BRK/repos/wfs-model/pull-requests/1278/commits)
- [Additional Rollup steps by James](https://bitbucket.apak.delivery/projects/BRK/repos/wfs-model/pull-requests/1305/diff#src/main/java/db/migration/common/V26_5_0_18_2__portRollupJobs.java)
- [Another set of changes by James around this](https://bitbucket.apak.delivery/projects/WFS/repos/wfs/pull-requests/17629/diff#wfs-graphql/src/main/resources/application.yml)
- [Steve original raised ticket](https://jira.apak.com/browse/ETPS-4630)

### SQL I used on pgadmin
```sql
select collectednetttiming, collectedtaxtiming, paidoutnetttiming, paidouttaxtimin from financialitem 
LIMIT(10)

select * from schema_version 
where script like '%26_5_0_18%'
--- sql files

select * from schema_deletion_version
delete * from schema_deletion_version WHERE version='26.5.0.18.3'

---data_update_version is java files
select * from data_update_version WHERE version like '26.5.0.18%'
delete from data_update_version WHERE version='26.5.0.18.2'


select * from financialItem

select 
id, description, creationdate collectednettrolluptiming, 
assetpopulatorreference, collectednetttiming, 
collectedtaxrolluptiming, collectedtaxtiming, 
paidoutnettrolluptiming, paidoutnetttiming, paidouttaxrolluptiming 
from financialitem
WHERE collectednetttiming NOT like 'WITHIN_ROLL_UP'
AND assetpopulatorreference NOT like 'EXPERIAN'

select * from financialitem where id = 15161
select * from financialinterest where id = 15161
select * from fiholder where financialitem_id = 15161
select * from financeproduct
where id = 14994


ALTER table financialitem add collectednettrolluptiming varchar(255)
ALTER table financialitem add collectedtaxrolluptiming varchar(255)
ALTER table financialitem add paidoutnettrolluptiming varchar(255)
ALTER table financialitem add paidouttaxrolluptiming varchar(255)

-- ALTER table financialitem rename collectednetttiming to collectednettrolluptiming
-- ALTER table financialitem rename collectedtaxtiming to collectedtaxrolluptiming
-- ALTER table financialitem rename paidoutnetttiming to paidoutnettrolluptiming
-- ALTER table financialitem rename paidouttaxtiming to paidouttaxrolluptiming

update financialitem set collectednetttiming='MONTHLY'
update financialitem set collectedtaxtiming='MONTHLY'
update financialitem set paidoutnetttiming='MONTHLY'
update financialitem set paidouttaxtiming='MONTHLY'

select * from alert

select * from recovery_schema_version
select * from data_update_version

SELECT CURRENT_DATE, CURRENT_TIME;

select * from financialinterest where type='REGISTRATION_AMEND'
delete from financialinterest where id=52407
```