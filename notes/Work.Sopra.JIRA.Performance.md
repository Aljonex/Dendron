---
id: t5tok8saxd76anl8a7r2bsq
title: Performance
desc: ''
updated: 1683720200092
created: 1683630761473
---
# SX-65126
## Slow generation of Loan Movement Summary Report
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
After a brief convo with Michael he sussed out it was [C74](https://c74-bausupport.apak.delivery/WFS/portal.faces) (this was in the client code of the ticket) and then I could generate the report with their data, then use the *Thread Dump Recovery Tool* in *System Admin, the first pass took 3minutes and 40seconds to generate.

To get into those environments otherwise download the repos `acc-dev-machine-conf` and `asp-dev-machine-conf` (this one more often), generally don't touch ACC or LIVE environments, and then add yourself in the users.yml and push to remote. Then you should get email with logon, if issues then change the sequence number.

If you change `core.conf` you need to rebuild, otherwise you can change local.conf you can just restart tomcat, something like `show_sql` can `ctrl+shift+f` for and find in a spring xml or something similar and then add the full string to `local.conf`.

With thread dumps its a dump of all the things that are happening at that given time, so best to copy and paste into notepad and do multiple before process ends and then find.

### Process with Jack
*Kibana* - useful system for looking at logs of all systems, live, dev environments and otherwise, can look specifically for environment and maybe my username, change times etc

Try and get a snapshot release out (ask Chris).
For mine, the version was 8.47.220.37, so I branched off of that, another way to do it may be to do multiple thread dumps again and find kind of hotpoints where things mix together. For this there is `DailyReportScriptedDataSetPage1` and 2 and 3, so add some high level logging, then get a bit more specific on methods and find where timings get a bit slower and try and be as specific as possible, use the TimeRecord object like Jack showed me in the `FeedSingleActionProcessor`. With those hotpoints 


```java
 TimeRecord tr = new TimeRecord();

FeedSingleActionProcessedResult feedActionProcessedResult = processSingleAction(actionElement, feed, handler);

LOGGER.info("{} - Processed action {} in {}ms", FeedLoggingFormatProvider.getFormattedString(feed), handler.getClass().getSimpleName(), tr.stop());
```
In reports there's layout and scripted dataset, and scripted dataset gets put in report layout. 

### First step
Made 7 thread dumps while the report generated, the thread that was involved on the Loan Movement Summary Report was `pool-3-thread-1`, from using `ctrl+f` to find that, I could find points of interest across each instance of the thread dump.
- `UnitReportDAO.getDailyStockMovementNonLiveOpeningBalance` and `UnitReportService` & `ReportService`
- `UnitReportDAO.getStockTransferMovementsOnDate` and for `UnitReportService` & `ReportService`
- `DailyReportScriptedDataSetPage1`
    - `.generateSummaryRowsForNonLiveLoans`
    - `.query`
    - `.getStockMovementsForDate`
    - `.getDailyMovementReconciliations`
    - `.getLiveReconciliations`
    - `.generateSummaryRowsByBusinessProcessForLiveLoans`
    - `.getStockTransferMovementsOnDate`
- `UnitReportDAO.getStockMovementsOnDate` and for `UnitReportService` & `ReportService`
- `UnitReportDAO.getStockTransferMovementsOnDate` and for `UnitReportService` & `ReportService`
- When done there's `UnitReportDAO.getUnitLevelReportResults` and for `UnitReportService` & `ReportService`

In `DailyReportScriptedDataSetPage1.java` in the private `query()` method there's an *if*-statement
```java
if (isGroupingByOrganisation) { ... }
```
And I was informed to check how this is set (defaults in class as false at top level but) via a bean `ctrl+shift+f` and change file mask to *\*.xml* and can see in `ReportContext.xml` it is by default set to true for `DailyReportScriptedDataSetPage` 1,2, & 3

### Build Snapshot version
You will need to trigger this job with your feature branch to build the release

https://jenkins.apak.delivery/job/WFS_Snapshot_Build/

Then since it is a cloud environment you will need to trigger this one to build the docker image

https://jenkins.apak.delivery/job/wfs-docker/

- To build you must do the snapshot first using `Build with Parameters` and put the snapshot version then maybe add something meaningful like the date on the end, it will fail if it ends or starts with a 0.
- Then you must wait until this is finished and then click master on the second link, then use the build with parameters and put in the same snapshot version, i.e. because I made this release first on the 10th of may I used `8.47.220.37.10.5.23`

 Had an issue building and Chris said if I see `[ERROR] javac: invalid flag --release` it means the java version is wrong, what I needed in the pipeline snapshot build:
- GIT_BRANCH: feature/sx-65126-investigate-loan-summary-report-off-tag
- SNAPSHOT_VERSION: 8.47.220.37.10.5.23
- JAVA_VERSION: openjdk 11

Then for second step make the VERSION: `<version.additional>-SNAPSHOT`

After this builds, ask all users of the environment if its okay to update the release and let them know the changes that were made and what (if anything) it could break.

Then proceed to the next step.

Repositories -> Sandbox : definitions of all cloud environments
- Resources: database credential stuff, experian is asset evaulation stuff
- Applications: basic definitions of environments
    - change version of WFS
    - `conf` : is like local conf for the env
    - basically change tag to my version, for this is c74-bausupport
    - edit in the UI if needs be, commit and create PR and once ticked wait for jenkins, add -SNAPSHOT to the end of version

Go into argo, sandbox applications then synchronize it `argo.apak.delivery/applications` 
### Kibana logs
Before adding logging there were only 3 logs being shown, when the LOAN_MOVEMENT_SUMMARY_REPORT started, then saying something to do with `LOANMVMT` and then the report finishing, so logs were added to commonly seen methods throughout the generation in the `System Admin -> Recovery Tools -> Thread Dump Recovery Tool` by repeatedly loading this and copying the contents across to a notepad file to see what was being called throughout.

Ideal is to add the highest level possible, so for example do DailyReportScriptedDataSetPage1, 2, and 3, start with that and drill into service and just say do the service layer stuff, then once you know where the time has come from drill into the DAO.
