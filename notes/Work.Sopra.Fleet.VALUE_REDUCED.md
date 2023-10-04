---
id: n40d3pf3nf5e6cd6qll2ybr
title: SX-65619 VALUE_REDUCED
desc: ''
updated: 1695396706312
created: 1693904405938
---
## ETPS-2148 Fees affecting funding amount
## SX-65619

### Refinement for 29/6 (GAP003)
- [Jira ticket](https://jira.apak.com/browse/ETPS-2148)
- [Jira SX](https://jira.apak.com/browse/SX-65619)
- [PR](https://bitbucket.apak.delivery/projects/WFS/repos/wfs/pull-requests/29850/overview)

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

### ***WFS-CORE***
#### Batch.Merge.Invoice
- `InvoiceMerge` ~ DONE
    - `#addSubItems` - *line 2109* looks like an OR clause will be needed fort new functionality also
    - `#iterateSubItems` - *line 2524 & 2542* add to description, and include in total
    - **Test** - `#setUpMockFinancialItem(ItemType, BigDecimal)` *line 2462* - add check here for `VALUE_REDUCED_FROM_PARENT` ???????????????? No need to duplicate, tests nothing to do with functionality

#### Controllers
##### Agreement controllers
- `AgreementService` ~ DONE
    - `#loadLoanAmendRequest(IItem, IBusinessProcessContext, boolean)` - *line 2066* - as above add OR clause if it is new effect
    - `#loadLoanFinancialRequest(IItem, IBusinessProcessContext)` - *line 2084* as above as both of these are for loading the whole invoice when it is these sub items
##### Interest Controllers
- `InterestCalculationPaymentSourceResolver` ~DONE
    - `#hasEffectOnFundingValueAddedToParent(IItem)` - *line 56* check to see if it affects funding value added to parent, either another boolean for `hasEffectOnFundingValueReducedFromParent` or change this to `hasEffectOnFundingValueChangedOnParent` and this alteration reflected in the filter where the `EffectOnFundingType` checked against is a set containing both `VALUE_ADDED_TO_PARENT` and `VALUE_REDUCED_FROM_PARENT` 
- `InterestCalculationPaymentSourceResolverTest` ???????????????? Unnecessary as doesn't test functionality of the new enum
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
- `FinancialItemValidatorTest` !!!!!!!!!!!!!!!!!!!!! DON'T DO - James R said
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
#### How pieces fit together

### Steps to see government subsidy being added
- First make it available on the `Administration -> Plan Configuration -> Finance Products` - I made it on the `Fleet Model World` product, under additional items, and new then just followed basic steps to set it up
- Then make it available on the `Administration -> Plan Configuration -> Finance Profiles` - I then added this to `FLEET_MODEL_WORLD_PROFILE`, under additional items, ticked it, selected it and saved
- Then make it available on the `Administration -> Plan Configuration -> Finance Plans` - then I did as above but in the Fleet01 plan, this links to the `fiHolderDetails.xhtml` page and subsequent controller
- Then navigate to Fleet Create Loan on Loan Management, and you should see it as long as you're on the same Plan you enabled it on.


### Meeting Acceptance Criteria
#### 1 ~ Setting Available
- This was about adding the enum to the model, then to format it nicely had to look at what was being checked in `div_financialItemDetails.xhtml`
- Could see all the others were nicely formatted and there was a method in the `financialItemDetailsController` called `getEffectOnFundingSelectItems()` and this used the MessagingManager, which briefly touches upon using translations to get the relevant translation
- So then `ctrl+shift+f` and search for `VALUE_ADDED_TO_PARENT`
- Add the translation both to `translations_en_e` (for EffectOnFundingType_VALUE_REDUCED_FROM_PARENT) amd then in `sysx_en.properties`
- need to load this change into db before this will update correctly

#### 2 
- Followed suit from 1

#### 3
- Value doesn't get subtracted when govt subsidy
- Turns out FinancialTransactionUtils wasn't only important thing
- Line in InvoiceMerge 2109 - `addSubItems` in this check we'll have to differentiate between VALUE_ADDED_TO_PARENT and VALUE_REDUCED_FROM_PARENT and take from the total cost
- Look in ItemService, this is where the `getTotalFundedDisplayValue` is and is how it gets calculated
- Keep an eye on `BaseLoanDetailsController` and `StreamlinedController`
```html
div_loanDetails.xhtml

<f:facet name="footer">
    <h:outputText value="#{controller.selectedInvoiceWrapper.totalFundedDisplayValue}"/>
</f:facet>
```

#### 4


#### 5


#### 6


### Solution
```java
public void validateScheduleTotals(final IItem item, final IScheduleProfile scheduleProfile) throws CoreException {
        LOGGER.info("Validate schedule totals");
        if (scheduleProfile != null && item.getBusinessProcessRequest() != null) {
            final ValidationException ex = new ValidationException(scheduleProfile, "entityValidation");
            if (EnumSet.of(ItemType.AMORTIZED_LOAN, ItemType.COST).contains(item.getItemType())) {
                validateSchedulePayments(item, scheduleProfile.getAmortizedSchedule(), ScheduleType.AMORTIZED, ex,
                        null);
            } else if (itemService.getPlanForItem(item)
                    .getSystemProduct()
                    .getSystemProductType() == SystemProductType.COMMERCIAL_LOAN
                    && item.getItemType() == ItemType.LOAN) {
                validateCommercialLoanTransactionsAreBalanced(item, ex);

            } else {
                final IFinancialItem financialItem = itemService.getFIHolderForItem(item).getFinancialItem();
                validateSchedulePayments(item, scheduleProfile.getCollectionNett(), ScheduleType.COLLECT_NETT, ex,
                        financialItem.getCollectedNettTiming());
                validateSchedulePayments(item, scheduleProfile.getCollectionTax(), ScheduleType.COLLECT_VAT, ex,
                        financialItem.getCollectedTaxTiming());
                validateSchedulePayments(item, scheduleProfile.getPayoutNett(), ScheduleType.PAY_OUT_NETT, ex,
                        financialItem.getPaidOutNettTiming());
                validateSchedulePayments(item, scheduleProfile.getPayoutTax(), ScheduleType.PAY_OUT_VAT, ex,
                        financialItem.getPaidOutTaxTiming());
                validateSchedulePayments(item, scheduleProfile.getRental(), ScheduleType.RENTAL, ex, null);

                if (SystemProductType.FLEET == itemService.getPlanForItem(item).getSystemProduct().getSystemProductType()
                        && hasGovernmentSubsidy(item)) {
                    BigDecimal subsidyNett = BigDecimal.ZERO;
                    for (IItem subitem : item.getItems()) {
                        if (subitem.getFinancialItemHolder().getFinancialItem().getEffectOnFunding() == EffectOnFundingType.VALUE_REDUCED_FROM_PARENT) {
                            subsidyNett = subsidyNett.add(subitem.getNettCost());
                        }
                    }

                    BigDecimal nettMinusDeposit = item.getNettCost().subtract(item.getDepositValue());
                    BigDecimal totalNettMinusDeposit = nettMinusDeposit.add(subsidyNett); //subsidy is negative nett value
                    if (BDMath.isStrictlyPositive(item.getNettCost()) // Asset has positive nett value
                            && (totalNettMinusDeposit.compareTo(BigDecimal.ZERO) <= 0 && subsidyNett.compareTo(BigDecimal.ZERO) != 0)) { //The expected total when calculated is (Total - deposit - subsidy) is above 0
                        rejectFleetFundingAmountError(item, ex, ScheduleType.COLLECT_NETT, totalNettMinusDeposit);
                    }
                }
            }
```
The last if statement for doing the required checks that were proposed in the Acceptance criteria. 
Otherwise, changes were made to all if statements looking for `VALUE_ADDED_TO_PARENT` by checking if `getEffectOnFunding().affectsParent()` which was a method added in the enum.

### Testing
No tests for/no relative tests for VALUE_ADDED_TO_PARENT:
- `LoanAddItemController` 
- `FinancialItemDetailsController`
- `BusinessProcessButtonUtils` (no relative tests for VALUE_ADDED_TO_PARENT)
- `CommercialLoanSearchController` (no relative tests for VALUE_ADDED_TO_PARENT)
- `LoanAmendController`
- `InterestAmendController`
- `ChangeVRMController`
- `ReversalProcess`
- `PaymentProcessHandlingValidator`
- `NewAgreementProcess`
- `LoanAmendProcess`
- `ItemStatusProcess`
- `ItemPaymentUtils`
- `AgreementBaseBusinessProcess`


Tests were there for `VALUE_ADDED_TO_PARENT` in test classes for:
- `AgreementSearchController` - added new test cases where needed
- `ItemViewController`
- `CommercialLoanViewController`
- `BusinessProcessCommon`


## Notes on solution
These notes were made once the PR was up and by going sequentially through the files involved in the PR and explaining the changes made, obviously as above the task was to create an additional item for things like government subsidies that took away value from the asset (reduced it), use cases would be things like subsidies for electric vehicles.

### Files Edited
- `ErrorTranslationDictionary.csv`
- `Translations_en_e.xml`
- `InvoiceMerge.java`
- `AgreementService.java`
- `InterestCalculationPaymentSourceResolver.java`
- `InterestCalculationService.java`
- `ItemService.java`
- `FIHolderValidator.java`
- `FinancialItemValidator.java`
- `ItemValidator.java`
- `PlanValidator.java`
- model/rows # `ItemViewRow.java` 
- `DefaultCostBreakdownUtility.java`
- `IANewAgreementProcess.java`
- `InternalAccountBaseBusinessProcess.java`
- `FinancialTransactionUtils.java`
- `AbstractValueTransactionCalculator.java`
- `AgreementBaseBusinessProcess.java`
- `BusinessProcessCommon.java`
- `DealerSaleProcess.java`
- `ItemPaymentUtils.java`
- `ItemStatusProcess.java`
- `LoanAddItemProcess.java`
- `LoanAmendProcess.java`
- `NewAgreementProcess.java`
- `PaymentHandlingProcessValidator.java`
- `ReversalProcess.java`
- `ValueTransactionProcess.java`
- `WholesaleValueTransactionCalculator.java`
- systemx # `ErrorCodes.java`
- `util/ValidationHelper.java`
- `OnboardOrganisationService.java`
- `PaymentSourceDataProviderImpl.java`
- `sysx_en.properties`
- `sysx_errors_en.properties`

