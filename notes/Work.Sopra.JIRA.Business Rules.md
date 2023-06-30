---
id: lidrzlw1ndk6p4mio0ecg40
title: The System
desc: ''
updated: 1683626439868
created: 1681918813163
---
### SX-65058 Remove CreditLineIsValidRule - Business Rule
Dawid kindly pointed me to `systemx/controllers/alerts/planrules` and I found that (from Chris) it is actually called `CreditLineNotValid` due to translations happening.
- Removed rule from `BaseSystemX.xml`
- Deleted `CreditLineNotValid.java` and the relevant test
- Deleted the `OVERRIDE_CREDIT_LINE_NOT_VALID` rule in `UserRight.java`
- Removed `CREDIT_LINE_NOT_VALID` from `WfsRuleType.java` and `WfsRuleName.java` from `systemx/model/plan` directory
- From following the above trail removed `NO_VALID` from a case check in significant failures in `CreditCheckResponseCreator.java` and its test.
- Remove the reference to the bean deleted from `BaseSystemX` in the `businessRulesList` called `credtiLineNotValid`
- Remove the reference to the bean deleted from `BaseSystemX` in the `webservicesService` called `credtiLineNotValid` 
- Remove from `translations` `CREDIT_LINE_NOT_VALID` from `translations_en_c.xml` and `UserRight_OVERRIDE_CREDIT_LINE_NOT_VALID` from `translations_en_u.xml` and `WfsRuleName_CREDIT_LINE_NOT_VALID` from `translations_en_w.xml` and `WfsRuleType_CREDIT_LINE_NOT_VALID` from `translations_en_w.xml`
- Removed `UPDATE fieldDef SET viewPermissionAuth='ROLE_OVERRIDE_CREDIT_LINE_NOT_VALID' WHERE userRight=409;` from `wfs-migration/src/main/resources/sql_scripts/2016-T03-pre.sql`

Needed to write model step for this so that it removed this rule on live before pushing it. To revert changes use `git revert <commitHash>` and do it in reverse order, newest to oldest!
Model step needs flyway change [see](https://confluence.apak.com/live/display/WIKI/Flyway%3A+Pending+Migrations), where the rule would be removed from the database via a new sql file that removes it from relevant tables.

[Example](https://bitbucket.apak.delivery/projects/BRK/repos/wfs-model/pull-requests/2157/diff#src/main/java/com/apakgroup/systemx/model/api/rates/IRateRule.java)

### SX-65126 Slow generation of Loan Movement Summary Report
Firstly is the *Loan Movement Summary Report EXCEL* **Daily Report** button enabled or not?
I had to recreate the steps in the ticket which were:

    Log into the system and select Dealing Company<br>
    Navigate to Reports > Real Time > Reporting<br>
    Under Daily reports, click Loan Movement Summary Report EXCEL<br>
    Time how long the report takes to generate.<br>
    Report should generate much faster after the fix is applied.

I realised quickly I didn't have the summary report and after a quick message in the 8.47 chat there were 2 options, either the stock movement report (which seemed viable) but then Archana and Jack demonstrated it was an option but needed enabling.

To enable this you should first login as the AdminPFC user then via `System Admin -> Initial Setup -> Configuration Admin` (for whichever Dealing Company) and scroll to Dealing Company reports and click the Daily Dc Reports dropdown and select Loan Movement Summary Report. And had to select it under UI Reports to display it on the system.

Created a Loan via loan management, the only hiccups were how to navigate options, and also a VIN (vehicle identification number was needed) so I generated one and it was `2HHYD28618H200871` (must be 17 digits).

To do this I had to first check the AC, which is as follows

    GIVEN: The Loan Movement Summary Report is enabled<br>
    WHEN: The Loan Movement Summary Report is generated in Realtime reporting<br>
    THEN: The report is generated in a reasonable time frame given the size of the data output 

Spent ages trying to recreate locally, lots of errors to do with the fact I was getting no NettCost but then Chris informed me that due to the volume of data it was best to use a cloud test machine.
After a brief convo with Michael he sussed out it was (C74)[https://c74-bausupport.apak.delivery/WFS/portal.faces] (this was in the client code of the ticket) and then I could generate the report with their data, then use the *Thread Dump Recovery Tool* in *System Admin, the first pass took 3minutes and 40seconds to generate.

To get into those environments otherwise download the repos `acc-dev-machine-conf` and `asp-dev-machine-conf` (this one more often), generally don't touch ACC or LIVE environments, and then add yourself in the users.yml and push to remote. Then you should get email with logon, if issues then change the sequence number.

If you change `core.conf` you need to rebuild, otherwise you can change local.conf you can just restart tomcat, something like `show_sql` can `ctrl+shift+f` for and find in a spring xml or something similar and then add the full string to `local.conf`