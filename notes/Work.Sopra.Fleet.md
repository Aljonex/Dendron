---
id: 79is3g8ias18k3ldsfd54r0
title: Fleet
desc: 'A file for knowledge on stuff to do with Fleet/Johto project'
updated: 1693997879456
created: 1686919376670
---
# Basics
The key aspects of Fleet that we're focusing on for now are going to be:

- Implementing changes to enable loan repayment schedules to be calculated using the Freehand API (this is another sopra product we'll be interacting with) and to capture the information needed from the user during loan creation to send to Freehand.
- Adding a new asset populator for CAP Gold, this is basically just adding a new way to populate details about an asset automatically from an identification key such as a VIN, which hits the CAP Gold
- Adding metrics + reporting for tracking CAP Gold usage

> Things of note: PPP, Asset Populators and Loan Creation

# Upgrade fleet sandbox
[Go to AWS EKS Non-prod eu-west-1/i1](https://bitbucket.apak.delivery/projects/EUW1NP/repos/i1/browse/applications)
- Edit the relevant yaml file, in this case `fleet-test-sandbox.yaml`
- Change the image to the version of WFS you want

## Product, Profile, and Plans
[Confluence page](https://confluence.apak.com/live/pages/viewpage.action?spaceKey=WIKI&title=Product%2C+Profile+and+Plans)

These store all the information on how a *loan/item* is created in SFP-W/WFS.
To create a plan, a *finance product* must be created followed by a *finance profile* then a *finance plan*:
- the product is where the base data is set like *Financial Items, Charging Profiles and Schedules*
- Once the base data is created and linked to the product a profile can be made and you select which entities are available for the profile.
- The plan is then created using the profile and the entities available from that profile, its where you select which *schedule/charging profile* is used by a financial item when required

## Financial Items
Defines how each item under an *Agreement Invoice* is setup, you will usually have one **main item** (like an *Asset*) and one *Interest Item* underneath the *Agreement Invoice*.
You can set loads of parameters for the item, some more common settings are:
- Asset
    - *Valuation and Price Percentage* - how much of valuation or price is funded by dealing company
    - *Item value sourced from* - where to get cost of asset from
    - *Item cost calculation* - how to calculate split of cost of asset between **gross, nett, tax**
    - *Effect on Funded Value* - what values are used for funded amount in interest calculation
    - *Process inclusions* - which processes this is valid for
    - *Payments*
        - *Paid out to/Collect from* - which organisation money is paid out to and collected from
        - *Nett/Tax timing* - when money is paid out/collected
    - *Business rules* - which business rules to use (only available at **plan level**)
    - *Schedules* - which schedules to use for payout/collection (only available at **plan level**)
- Interest
    - *Payments* - same as asset but should only have collected from settings
    - *Charging Profile* - the charging profile to use to calculate interest (only available at **plan level**)
- Additional Items
    - *Item value sourced from* - where to get cost of additional item from
    - *Default fee* - default/fixed amount
    - *Item Single/Multiple* - how many times fee can be raised
    - *Process inclusions* - which business processes this is valid for
    - *Payments* - same as interest
    - *Effect on funded value* - what values are used for funded amount in interest calculation

## Charging Profiles
This defines how interest gets calculated through the life of the loan, you can set who is responsible for paying the interest at which stage and the rate the interest gets calculated at. 
A charging profile contains one or more Charging Timings, these define each interest period and how its calculated.

- Charging Profile
    - *Interest Calculator* - which calc to use for the interest calculation, typically varies by number of days of the year
    - *Predicted/Charging Calculation Timing* - when interest is calculated and how far in advance predicted interest is calculated
- Charging Timing
    - *Party* - who pays interest in the period
    - *Start/End type* - when period will start and end
    - *Type* - can be **fixed, nil, or variable**
        - *fixed* = rate stays the same
        - *nil* = rate is always 0
        - *variable* = rate reflects any changes in Rate
    - *Base Rate* - base rate to use for interest
    - *Rate* - any rate applied on top of base
    - *Min/Max Rate* - caps for min and max that can be applied
    - *Rate Reduction* - whether rate is reduced based on band selected

## Schedules
Defines when transactions are due to be paid and how much each payment is, can either be percentage or value based:
Common settings include:

- *Schedule Type* - which type of payment can happen, **payout or collection and nett or tax**
- *Calculation Type* - either percentage or value, deteremines if payments are fixed or percentage of total
- *Driven by* - days, months, or date
- *Payment Skipped Type* - if payment is changed/removed where should remaining amount go, could be immediate payment or adjustment of final payment
- *Payments*
    - *Day/Month/Date* - when payment is due
    - *Percentage/Value* - value of transaction
    - *Transaction type* - must always be final transaction at end but ones preceding this can be **Adjustment or Normal**

In the UI go select a Dealing company and use `Administration -> Plan Configuration`

### Key Classes
- `ProductProfilePlanService` - service class for all of the entities, includes methods to retrieve and save information from the database
- `PlanDetailsController` - UI Controller for plan viewing and editing
- `FinanceProfileDetailsController` - UI Controller for viewing and editing finance profiles
- `FinanceProductDetailsController` - UI Controller for viewing and editing finance products


## ETPS-2583 Add Fleet Finance Product to ModelWorld (SX-65481)
Useful files
- `pppExtract.xml` - where the data gets loaded into db during `FeatureTestsRunner.runFeatureTests()`
- `AutoMigrateService.java` where `#exportDcZip()` hides for exporting the modelworld data for the given env

Tim suggested making use of the ScriptRunner, so to do this I wanted to first test locally. 
What I hadn't realised is that the ScriptRunner runs on *Groovy* scripts, which are effectively Java code.
I found on confluence an example of a groovy script and pieced together my own after reading through the `AutoMigrateService#exportDcZip()` code and landed with:
```Java
import com.apakgroup.systemx.controllers.automigrate.AutoMigrateService;
import com.apakgroup.systemx.dao.org.OrganisationDAO;
import com.apakgroup.systemx.model.api.org.IDealingCompany;
import java.io.File;  

AutoMigrateService autoMigrateService = context.getBean("autoMigrateService");
File file = new File("C:\\temp\\migration-data\\exportZip.zip");
autoMigrateService.exportDcZip( file, 12658, null, true, true, "8.47.499", null, null);
```

Tim pointed me to this [useful test](https://bitbucket.apak.delivery/projects/WFS/repos/wfs/browse/wfs-core/test/com/apakgroup/systemx/controllers/automigrate/AutoMigrateServiceIT.java?at=support%2F8.55%2Fdev#92) (only used on 8.55 hence not being able to see it on 8.47) so that I could potentially make a script to output my plan.xml.

Issue populating WFS db with `FeatureTestRunner#runFeatureTests()` from James R's `feature/etps-2141-fleet-prototyping`, checked out this branch on WFS-model also.
Clean building wfs-model repo was causing issues because I imported my run configs from `wfs` which had `clean package -pl wfs-ui -am -DskipTests -T1C` in Run section, and the wrong working directory.
Needed to change working directory to `wfs-model` and run args to `clean package -am -T1C`. 
Was still getting an error with `Cannot resolve symbol 'FLEET'` but using the Maven tab and reload all projects button resolved it.

```Java
import java.io.File;  
import org.apache.commons.io.FileUtils;
import javax.faces.context.FacesContext;
import javax.servlet.http.HttpServletResponse;
import java.util.zip.ZipFile;
import org.apache.commons.io.IOUtils

import com.apakgroup.systemx.controllers.automigrate.AutoMigrateService;
import com.apakgroup.wfs.base.util.FileUtil;

Set<String> constituentFileSet = new HashSet<String>();
constituentFileSet.add("plan");
File exportZipFile = new File(FileUtil.getTempDir() + "TEST" + "export.zip");
AutoMigrateService autoMigrateService = context.getBean("autoMigrateService");
autoMigrateService.exportDcZip( exportZipFile, 12755, null, true, false, "wfs", "test", null);

FacesContext facesContext = FacesContext.getCurrentInstance();
HttpServletResponse response = (HttpServletResponse) facesContext.getExternalContext().getResponse();
response.setContentType("text/xml");
response.setHeader("Content-Disposition",
		"plan.xml");
OutputStream output = response.getOutputStream();
//Read zip as input stream using zip entry
ZipFile zipFile = new ZipFile(exportZipFile.getPath());
String fileName = "plan.xml"
IOUtils.copy(zipFile.getInputStream(zipFile.getEntry(fileName)), output);


FacesContext.getCurrentInstance().responseComplete();
```
This script worked locally. It then also worked on the cloud env.
The first 3 lines came from Tim's useful test he sent me, the `12755` came from making use of `select * from organisation where dtype='DealingCompany'` in the `SQLRunner` to find the id of the Dealing Company.
ZipFile things came from google searching and a chat with Tim around [this test of zip parsing of ageing data](https://bitbucket.apak.delivery/projects/WFSU/repos/wfs-ageing-test-framework/pull-requests/132/diff#wfs-ageing/src/main/java/com/apakgroup/wfs/modelworld/util/AgeingFileUtil.java?t=13).
The lines from `FacesContext` to `OutputStream` came from `UIReportGenerator.java`
``` java
/**
 * Stream the content of the report to the HTTP response.
 * 
 * @param report
 *            The entry with the details of the report to be streamed.
 */
public void streamReportToBrowser(IReportQueueEntry report) {
    try {
        reportQueueService.loadReport(report,
                getResponseOutputStream(report.getFormat(), new File(report.getOutputURI())));
        FacesContext.getCurrentInstance().responseComplete();
    } catch (IOException e) {
        LOGGER.warn("Could not load the report {}", report, e);
    }
}
private OutputStream getResponseOutputStream(ReportGenFormat reportFormat, File file) throws IOException {
    FacesContext facesContext = FacesContext.getCurrentInstance();
    if (!facesContext.getResponseComplete()) {
        HttpServletResponse response = (HttpServletResponse) facesContext.getExternalContext().getResponse();
        response.setContentType(ReportFormat.valueOf(reportFormat.name()).getContentType().value());
        response.setHeader("Content-Disposition",
                ReportFormat.valueOf(reportFormat.name()).getHeaderValue(file.getName()));
        return response.getOutputStream();
    }
    return null;
}
```
Then finally the `IOUtils.copy` came from `ReportQueueFileSystemStorage.java`
```java
@Override
public URI storeReport(IReportQueueEntry entry, InputStream contents) throws IOException {
    File file = getFileToBeWritten(entry);

    try (OutputStream fileStream = new FileOutputStream(file)) {
        IOUtils.copy(contents, fileStream);
    }
    return file.toURI();
}
```

In the end I had to override the `pppExtract` and `miscExtract` xmls, in `wfs-core/resources/configuration/mw_lite_configuration/DC_1/platinum_data` with ones extracted from the sandbox test env, which had a combination of normal model world and fleet data loaded, then work out some kinks of copies, had to add both as `miscExtract` deals with new ItemDescriptors and `pppExtract` deals with everything Product, Profiles, and Plans related.

