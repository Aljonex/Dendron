---
id: 79is3g8ias18k3ldsfd54r0
title: Fleet
desc: 'A file for knowledge on stuff to do with Fleet/Johto project'
updated: 1688742075578
created: 1686919376670
---
# Basics
The key aspects of Fleet that we're focusing on for now are going to be:

- Implementing changes to enable loan repayment schedules to be calculated using the Freehand API (this is another sopra product we'll be interacting with) and to capture the information needed from the user during loan creation to send to Freehand.
- Adding a new asset populator for CAP Gold, this is basically just adding a new way to populate details about an asset automatically from an identification key such as a VIN, which hits the CAP Gold
- Adding metrics + reporting for tracking CAP Gold usage

> Things of note: PPP, Asset Populators and Loan Creation

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

## ETPS-2148 Fees affecting funding amount

### Refinement for 29/6 (GAP003)
[Jira ticket](https://jira.apak.com/browse/ETPS-2148)

Basics from James R:
- They want to be able to make *Additional Items* reduce the funded value of the asset
- `EffectOnFundingType.VALUE_ADDED_TO_PARENT` exists
- We want to add `VALUE_REDUCED_FROM_PARENT`
- Start by looking at where the value is being calculated to add the case to reduce it when new enum option is set

#### Questions
- A finance house user is a dealing company?
- Is a government subsisdy the only thing reducing funded amount to customer?
- Why does a Road Fund license increase funded amount to customer?
- By single repayment schedule "The system shall allow customers to repay the the total funding amount (Asset Funding Amount + Road Fund License - Subsidy) with a single repayment schedule." does this mean the total cost just gets calculated and put into a single payment plan
- Where should fees affecting funding amount be configurable?
- I'm unsure of point 5.0 on system requirements, if the subsidy made the funding amount be negative surely that would be a huge amount as the whole thing goes on a repayment plan, how likely is this? Fine if its just a check but just making sure I'm understanding this
- Missing prerequisite loan setup?
- First 5 AC seem very informative of what is wanted

### Dev Approach 
Go to `EffectOnFunding` enum class and find all usages of the `VALUE_ADDED_TO_PARENT` since this `VALUE_REDUCED_FROM_PARENT` is conceptually very similar, so can find out what should be simple and what you're not sure about (84 usages):
- Firstly looks like it will be `VALUE_REDUCED_FROM_PARENT(EffectOnValueType.NONE)`

***WFS-CORE***
#### Batch.Merge.Invoice
- `InvoiceMerge` ~ DONE
    - `#addSubItems` - *line 2109* looks like an OR clause will be needed fort new functionality also
    - `#iterateSubItems` - *line 2524 & 2542* add to description, and include in total
    - **Test** - `#setUpMockFinancialItem(ItemType, BigDecimal)` *line 2462* - add check here for `VALUE_REDUCED_FROM_PARENT` ????????????????

#### Controllers
##### Agreement controllers
- `AgreementService` ~ DONE
    - `#loadLoanAmendRequest(IItem, IBusinessProcessContext, boolean)` - *line 2066* - as above add OR clause if it is new effect
    - `#loadLoanFinancialRequest(IItem, IBusinessProcessContext)` - *line 2084* as above as both of these are for loading the whole invoice when it is these sub items
##### Interest Controllers
- `InterestCalculationPaymentSourceResolver` ~DONE
    - `#hasEffectOnFundingValueAddedToParent(IItem)` - *line 56* check to see if it affects funding value added to parent, either another boolean for `hasEffectOnFundingValueReducedFromParent` or change this to `hasEffectOnFundingValueChangedOnParent` and this alteration reflected in the filter where the `EffectOnFundingType` checked against is a set containing both `VALUE_ADDED_TO_PARENT` and `VALUE_REDUCED_FROM_PARENT` 
- `InterestCalculationPaymentSourceResolverTest` ????????????????
    - `#setupCollectFromParentOrgItem(IPaymentSource paymentSource)` - *line 231* building based on collection from parent item, probably want to check if its either type again
- `InterestCalculationService` ~ DONE (added REDUCED to enum_set)
    - `#populateTransactionPaymentSource(IItem fundedItem, EffectOnFundingType fundingType, FtType ftType)` - *line 1633* **unsure** what this one is doing
##### Item controllers
- `ItemService` ~ DONE
    - `#getChildItemsAddingToFunding(IItem item)` - *line 808* this is checking for list of child items contributing to funding of this, so surely subsidy should be included here
    - `#getBPRSubItemsAddingToFunding(IItem item)` - *line 824* similar to above, so should be added
- `ItemServiceTest` ~ DONE
    - `#testGetCurrentOutstandingValue()` - *line 1176* potentially should be added to new but very similar test as another holder to check reduction happens correctly
##### Validators
- `FIHolderValidator` DONE
    - `#validateSubItem(IFIHolder fiHolder, Errors errors)` - *line 263* seems this is a check of validation that this child has no children and the parent is valid, probably seems more reasonable to create a set for `REDUCED` and `ADDED` children, RFLs and subsidies
- `FinancialItemValidator` DONE
    - `validateOtherParameters(IFinancialItem financialItem, Errors errors, ValidationContext validationContext)` - *line 604* simple check parent holds all financial data other than cost, probably again, add OR check for `REDUCED`
- `FinancialItemValidatorTest` !!!!!!!!!!!!!!!!!!!!! DON'T DO
    - `#testValidateOtherParameters_WithItemsParentHoldingAllFinancialDataExcludingCost()` - *line 1302* - unsure how to add here,  seems maybe duplicate test or as above, set of enums that contains both RFL and subsidy
- `ItemValidator`  DONE
    - `#validateCostValues(IItem item, Errors errors)` - *line 470* - seems like add OR to the case as it is just iterating over parent values
- `PlanValidator` DONE
    - `#validateFIHolderOrgs(IPlan plan, Errors errors, IFIHolder fiHolder, ValidationContext validationContext)` - *line 380* seems like its just checking the main financial item holder isn't one of these additional items otherwise its sending an error

#### Model
##### Rows
- `ItemViewRow` DONE
    - `#isInvolvedWithParent(List<Long> orgDescendantIds)` - *line 132* checks if it is a subitem linked to its parent, OR statement or collect enums in set
    - `#getSignValue(BigDecimal value)` - *line 626* chceking its not of this EOF type, OR statement or bring both into enum set
- `ItemViewRowTest` DONE
    - `#testGetSignValueEffectOnFundingValueAddedToParentAndNotCollected()` - *line 56* - duplicate test but renamed and change value of EOF type

#### Process
- `AgreementBaseBusinessProcess` DONE
    - `#loadBusinessProcessRequest` - *line 476* OR case for new subsidy, as is initialising request for all items and excluding any non relevant, throwing an error
    - `#isAdditionalItemApplicableForRequestValidation` - *line 569* checking additional item, seems like these EOF types aren't applicable so another OR case
    - `#refreshSubItem` - *line 1876* another OR statement
- `BusinessProcessCommon` DONE
    - `#getStatus` - *line 2842* sets item status based on info at hand, OR statement for subsidy or contain in enum set
- `BusinessProcessCommonTest` DONE
    - `#testGetStatus_hasFinancialImpact` *line 1924* add `IFinancialItem valueReduceFromParentPlanItem` similar to the `valueAddToParentPlanItem`, find out need for financialItemBuilder `withPaidOut` etc values
    - `#testGetStatus_infoItem` - *line 1951* seems can use `EnumSet.of(EffectOnFundingType.VALUE_ADDED_TO_PARENT, EffectOnFundingType.VALUE_REDUCED_FROM_PARENT)`
- `DealerSaleProcess` DONE
    - `#copyFinancialData` *line 95* similar to above, check if contained in enum set 
- `DealerSaleProcessTest` !!!!!!!!!!!!!!!!!!!!!
    - `#buildInterestFinancialItem` *line 914* - 7 uses of this `IFinancialItem` so should probably make something that is a building a subsidy child item and adding to test cases
- `ItemStatusProcess` DONE
    - `#validItem` *line 339* just invalidating if its EOF type is the child type so should check instead if its in a set of something like `VALUE_AFFECTS_PARENT`
- `LoanAddItemProcess` DONE
    - `#isLoadSchedulePayments` *line 122* checking if its a sub item, again probably should make enum set for this
    - `#removeItem` *line 191* as above as this is removing child from parent then refreshing parent
    - `#refreshRequest` *line 218* probably as above
    - `#processRequest` *line 262* as above as is just setting boolean involving parent item to true if the child item is what is currently selected
- `LoanAddItemProcessIT` !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
    - `#testLoanAddItemMultiPartFundedWithDescriptors` *line 1653* this test uses an old reference that I'm not sure is used anymore, **ASK**
    - `#setupSubItemFundedFIForCCUK1` *line 3832* similar to above, using reference I've not seen before
    - `#setupCollectionScheduleForFundedItemsCCUK1` *line 3954* probably just check its part of enum set as this bit is just checking the EOF type isn't one of the child types
- `LoanAddItemProcessTest` !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
    - `#buildAssetFinancialItem` *line 860* helper method set in test class, used 15 times, probably want to build a similar helper using the builder but with new child type and add in other cases
- `LoanAmendProcess` DONE
    - `#selectItemsToProcess` *line 147* including parent items if the EOF type is one of these, include OR statement or check if part of enum set
- `NewAgreementProcess` DONE
    - `loadTransactions` *line 231* as above probably just check not part of set of these child items
- `NewAgreementProcessTest` !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
    - `#testLoadTransactionsWholesaleItemValueAddedToParent` *line 1325* likely make another test very similar simply where item reduced from parent
    - `#testRefreshSubItemValueAddedToParent` *line 1634* as above
- `ReversalProcess` DONE
    - `#loadBusinessProcessRequest` *line 82* throwing exception if child item, so again check OR or add enum set check against

##### Costbreakdown.mutators
- `DefaultCostBreakdownUtility` DONE
    - `#getEffectOnFunding` *line 43* - add OR to case, as is just getting parent items effect on value
    - `#getEffectOnUtilised` *line 61* as above
- `DefaultCostBreakdownUtilityTest` ???????????????????
    - `#testGetEffectOnFunding_WithValueAddedToParent` *line 116* dupe test with value reduced most likely
    - `#testGetEffectOnFunding_WithValueAddedToParentAndNoParentItemPresent` *line 139* as above
    - `#testGetEffectOnUtilised_WithValueAddedToParent` *line 196* as above
    - `#testGetEffectOnUtilised_WithValueAddedToParentAndNoParentItemPresent` *line 243* as above

##### Internal account
- `IANewAgreementProcess` DONE
    - `#loadTransactions` *line 168* NOT EQUAL, so check if part of `EnumSet.complementOf(ADDED, REDUCED)` maybe
- `InternalAccountBaseBusinessProcess` DONE
    - `#refreshSubItem` *line 680* as above

##### Util.FinancialTransaction
- `FinancialTransactionUtils` DONE
    - `#resolveItemNettTaxItemCostValue` *line 48* - this is where a case for `.subtract` needs to made, also need to consider what happens when the getNettTaxCost is greater than total cost and how to error this out
- `FinancialTransactionUtilsTest` 
    - `#testResolveItemNettTaxItemCostValueWithSubItemAddingToParent` *line 102* make a new test with value being reduced from the parent

***WFS-UI***
#### Agreement
##### Commercial
- `CommercialLoanViewController` DONE
    - `#isItemSettleable` *line 526* check its not part of enum set containing both VALUE_ADDED and VALUE_REDUCED
- `CommercialLoanViewControllerTest` !!!!!!!!!!!!!!!!!!!!!
    - variable set up, not sure if to duplicate or not as seems to just check a button working, imagine i should duplicate variable and do the same test but for the new enum

##### Item
- `ChangeVRMController` DONE
    - `#getScheduleItem` *line 104* add check for new enum type, again enum set would probably work
- `InterestAmendController` DONE
    - `#getScheduleItem` *line 63* as above
- `ItemViewController` DONE
    - `#isShowTransactionHistory` *line 622* checking for not equal, so check not part of set containing both VALUE_ADDED and VALUE_REDUCED
- `ItemViewControllerTest`???????????????????
    - `#testShouldReturnFalseGivenEffectOnFundingTypeIsValueAddedToParentWhenShowTransactionHistory` *line 351* duplicate test but for new enum type 
- `LoanAmendController` DONE
    - `#getScheduleItem` *line 805* as the top two as is getParentItem() call inside check

##### Search
- `AgreementSearchController` DONE
    - `#isItemInSettlableStatus` *line 1261* checking negation that EOF type is VALUE_ADDED, so create enum set to check against also containing VALUE_REDUCED
    - `#isItemReversable` *line 1315* as above
    - `#isItemDeconsignable` *line 1416* as above
- `AgreementSearchControllerTest` !!!!!!!!!!!!!!!!!
    - `#testIsItemDeconsignable_withDeconsignableRows` *line 879* add `deconsignableEffectOnFundingTypes.remove(EffectOnFundingType.VALUE_ADDED_TO_PARENT);` for VALUE_REDUCED
    - `#testIsItemDeconsignable_withNonDeconsignableRows` *line 891* duplicate test for VALUE_REDUCED
    - `#testCheckEachSelectedRowPermission_rowsThatAreNotDeconsignable` *line 985* dupe for VALUE_REDUCED
    - `#testIsShowDeconsignButton_oneRowSelected` *line 1014* unsure if I should remove VALUE_REDUCED_FROM_PARENT in the same method
    - `#testIsShowDeconsignButton_multipleRowsSelected` *line 1051* as above
- `CommercialLoanSearchController` DONE
    - `#isItemReversable` *line 259* checks if VALUE_ADDED and returns false, so most likely add check for part of enum set also containing VALUE_REDUCED 

##### Util
- `BusinessProcessButtonUtils` DONE
    - `#isItemReversable` - *line 65* checking a negation for EOF being VALUE_ADDED so add enum set check with VALUE_REDUCED contained inside

#### FinancialItem
- `FinancialItemDetailsController` DONE
    - `#isArrearsVisible` again is negation for EOF not being VALUE_ADDED_TO_PARENT so enum set seems valid here
    - `#isOffsetTypeVisible` as above
    - `#isClockStartsFromVisible` as above
    - `#isChargingStartsFromVisible` as above
    - `#isChargingStartsFromDelayDaysVisible` as above
    - `#isChargingPaymentDelayVisible` as above
    - `#isInsuranceStartsFromVisible` as above
    - `#isInsuranceStartsFromDelayDaysVisible` as above
    - `#isInsurancePaymentDelayVisible` as above
    - `#isInsurancePaymentDelayDaysVisible` as above
    - `#isInsuranceEndsOnVisible` as above
    - `#isInsuranceEndsOnDelayDaysVisible` as above
    - `#isPayoutChargingDelayVisible` as above
    - `#isPayoutChargingDelayTypeVisible` as above

#### LoanAmend 
- `LoanAddItemController` DONE
    - `#getScheduleItem` *line 800* add check for new enum type, again enum set would probably work as checking to then `getParentItem()`


seems can use `EnumSet.of(EffectOnFundingType.VALUE_ADDED_TO_PARENT, EffectOnFundingType.VALUE_REDUCED_FROM_PARENT)`

Chris Mobley did lots on NewAgreementProcess it seems

### Local Testing
Go into DC, create fleet loan, however there were no credit lines, meaning mistake from SX-65481, so pulled `credit.xml` 

The `creditLineDetails` is a `json` file however, so with the xml, assessed what were defaults and general things needed and the general layout is:
```json
{
        "dealerReference" : "DEALER1",
        "creditLines" : [
            {
                "reference" : "DEALER1_NEW_CAR",
                "name" : "New Credit Line",
                "validForNonSupplierLoans" : false,
                "limit" : "895000",
                "currency" : "GBP",
                "assetTypes" : ["CAR"],
                "status" : "OK",
                "calculationType" : "FIXED",
                "facilities" : [
                        "NEW"
                ]
            }
    ]
}
```
In the template certain things are set to defaults so if not used in xml they will be overridden, i.e:
- `"currency" : "GBP"`
- `"calculationType" : "FIXED"`
- `"validForAllSupplierLoans" : "false"`
- `"validForNonSupplierLoans" : "false"`

### Points of Interest
- `newLoan.xhtml`
- `agreementView.xhtml`
- `EffectOnFunding` - enum that has all values, need to add `VALUE_REDUCED_FROM_PARENT`
- `div_financialItemDetails.xhtml` - this is the page under `Admin -> Plan Configuration -> Finance Products` for the user and under the funding section will allow the addition of `VALUE_REDUCED_FROM_PARENT` as you can add the `VALUE_ADDED_TO_PARENT` here
- `FinancialItemDetailsController.java` - this is where `getEffectOnFundingSelectItems()` lives for selection of the new option, and there's most likely a means of calculation here.
- It appears calculation occurs in `FinancialTransactionUtils#resolveItemNettTaxItemCostValue` and hopefully the calculation would simply be another if statement for `VALUE_REMOVED_FROM_PARENT` where total cost would be set to `totalCost = totalCost.subtract(getNettTaxCost(subItem, nettTaxType));`
- In a few controllers there's something along the lines of:
```java
if (scheduleItem != null && itemService.getFIHolderForItem(scheduleItem).getFinancialItem()
                .getEffectOnFunding() == EffectOnFundingType.VALUE_ADDED_TO_PARENT) {
            scheduleItem = scheduleItem.getParentItem();
        }
```
So it looks like I'll need to add an `AND` statement checking for `VALUE_REMOVED_FROM_PARENT`
- Also **many** tests will need to be added to test through this

### Steps to see government subsidy being added
- First make it available on the `Administration -> Plan Configuration -> Finance Products` - I made it on the `Fleet Model World` product, under additional items, and new then just followed basic steps to set it up
- Then make it available on the `Administration -> Plan Configuration -> Finance Profiles` - I then added this to `FLEET_MODEL_WORLD_PROFILE`, under additional items, ticked it, selected it and saved
- Then make it available on the `Administration -> Plan Configuration -> Finance Plans` - then I did as above but in the Fleet01 plan, this links to the `fiHolderDetails.xhtml` page and subsequent controller
- Then navigate to Fleet Create Loan on Loan Management, and you should see it as long as you're on the same Plan you enabled it on.


### Meeting Acceptance Criteria
#### 1 ~ Setting Available
