---
id: pcky9r51t4nfi96m72feeb3
title: SX-67304 Gap118B
desc: 'Internal Account Dealer Statement'
updated: 1726150357451
created: 1708086637100
---
### Basics
Biggest ticket I've picked up to date, and the logical step after this is 118A which is blocked by this, estimated by Rich as 26 days, so most likely will take longer. To be broken into subtasks.
- Adding the report to be visible in UI **(1 day)** - simply configuring a report to appear in the UI as normal and then creating the report layout and scripted data sets necessary for a blank report to be produced. 
- Adding heading information and the account details **(2 days)** - should be a case of mimicking existing reports for the heading information, and the account details will just be the account number and identification for the internal account and the query to pull back the opening and closing balance (which we already have in C3InternalAccountDealerStatementService). Extra time required for writing integration test and adding the initial info to the report layout.
- Adding transaction summary **(2 days)** - should be a case of getting the transactions from the query provided in the dev approach. Extra time required for writing integration test and adding info to report layout. 
- Adding charges summary **(3 days)** - as this one has a slight bearing on performance, more consideration needs to be taken. Can utilise the query provided in DealerStatementService and then transform relevant rows or use the one in the dev approach. Some manipulation required at the service layer. Extra time required for adding the relevant info to the report layout and for writing IT.
- Adding interest breakdown summary **(2 days)** - just utilise the query provided and then time is required for adding IT and the relevant info to the report layout. 
- Generating as part of EOD **(3 days)** - time required to add the CachedReportGeneratorCondition and relevant IT. Will also need to hook up the config so it can be produced as part of the EOD. 
- Mulit-threading and caching **(10 days)** - now we could do this as part of a separate ticket, but would be good to get this done right from the beginning. We should cache the data at the very beginning and then we can build on that so it doesn't use up much of the time. The multi-threading, however, is likely going to take some time to implement and test. 
- Testing document and demo **(5 days)** - if we take a blanket rule of 5 scenarios per day, it's going to take around 5 days to provide all the testing evidence and demo the functionality. Will likely also take a couple of days to perform sanity testing and RBRG. 
- On startup don't forget to enable report in UI, Dealer, Dealing company and remember its monthly!

**Actually took 66 days end to end for first portion, then Rich did multithreading and that took 4 days testing** - it took me 2.69x longer than predicted

### Tips on UI nav
- Internal account search under `Administration -> Credit Management -> Internal Account Search`, need to switch off default of surplus account
- Can also view loans under stock list, need to remove the date filter
- Log into dealer using Account Enquiry -> Dealer Selection


### Chats with Rich
- Turns out my getInternalAccountDetailsAndOpeningBalance was causing issues and what I should be doing is follow a similar route to c3 where the query is run and opening and closing balances are added afterwards
    - There weren't utilised transactions for first of the month because first transaction was 29/01


### Points of note
- `.groupColumns()` vs `.dataColumns()` in ReportLayout levels, some data is necessary to be specified be it fields, group columns or data columns, not sure the differences though
- Last part of REQ4 involved using 2 different methods and making them work for me from DealerStatementService, both of them make use of `getBillingStatementBalanceData` from the `DealerStatementDAO` (the service methods in question were `getBillingStatementBalanceDataFromQueryData` and `getBillingStatementTransactionLevelData`) and the DAO method looks like (with comments explaining the old criteria working by Chris)
```Java
public List<InternalAccountDealerStatementReportRow> getBillingStatementBalanceData(final ReportingParameters rp) {
        DetachedCriteria crit = DetachedCriteria.forClass(BalanceTransaction.class, "obt");
        crit.createAlias("obt.organisation", "org");
        crit.createAlias("obt.item", "item");
        crit.createAlias("item.agreementInvoice", "agreementInvoice");
        crit.createAlias("item.statusHistory", "statusTransaction");
        //joins on tables, but in fields in the code, i.e. BalanceTransaction has IItem and IOrganisation
        ProjectionList list = Projections.projectionList();
        // These are just creating aliases
        list.add(Projections.distinct(Projections.property("obt.id")), "id");
        list.add(Projections.property("agreementInvoice.id"), "agreementId");
        list.add(Projections.property("agreementInvoice.reference"), "agreementReference");
        list.add(Projections.property("org.id"), "organisationId");
        list.add(Projections.property("org.reference"), "organisationReference");
        list.add(Projections.property("org.organisationType"), "organisationType");
        list.add(Projections.property("obt.endDate"), "endDate");
        list.add(Projections.property("obt.effectiveDate"), "effectiveDate");
        list.add(Projections.property("obt.value"), "value");
        list.add(Projections.property("obt.balanceType"), "balanceType");
        list.add(Projections.property("obt.status"), "status");
        list.add(Projections.property("item.identification"), "itemIdentification");
        list.add(Projections.property("item.id"), "itemId");
        list.add(Projections.property(PlanDAOUtils.getPlanIdJoin("agreementInvoice")), "planId");
        crit.setProjection(list);
        if (rp.getLocalCurrency() != null) {
            crit.add(Restrictions.eq("agreementInvoice.currency", rp.getLocalCurrency()));
            //These are just where clauses
        }

        if (balanceTypes != null && !balanceTypes.isEmpty()) {
            crit.add(Restrictions.in("obt.balanceType", balanceTypes));
        }
        crit.add(Restrictions.eq("obt.status", ItemStatusType.ACTIVE));

        if(!excludedStatuses.isEmpty()) {
            crit.add(Restrictions.not(Restrictions.in("statusTransaction.itemStatus", excludedStatuses)));
        }

        //Inner queries are from disjunctions or conjunctions, disjunctions is an OR case
        // Conjunctions are AND
        // So below is : A OR (B AND C AND (D OR E))
        Disjunction disj = Restrictions.disjunction();
        disj.add(Restrictions.between("statusTransaction.effectiveDate", rp.getStartDate(), rp.getEndDate()));
        // or still in a valid active status
        Conjunction activeWithinPeriod = Restrictions.conjunction();
        activeWithinPeriod.add(prepareNotInForExcludedItemStatus());
        activeWithinPeriod.add(Restrictions.lt("statusTransaction.effectiveDate", rp.getStartDate()));

        Disjunction nullOrFutureEndDate = Restrictions.disjunction();
        nullOrFutureEndDate.add(Restrictions.gt("statusTransaction.endDate", rp.getEndDate()));
        nullOrFutureEndDate.add(Restrictions.isNull("statusTransaction.endDate"));
        activeWithinPeriod.add(nullOrFutureEndDate);
        disj.add(activeWithinPeriod);
        crit.add(disj);

        crit.setResultTransformer(Transformers.aliasToBean(ValueTransactionRow.class));
        return findByCriteria(crit);
    }
```

Attempted conversion
```Java
public ImmutableList<ValueTransactionRow> getBillingStatementBalanceData(final ReportingParameters rp) {
        CriteriaBuilder cb = getCriteriaBuilder();
        CriteriaQuery<Tuple> crit = cb.createTupleQuery();

        Root<BalanceTransaction> obtRoot = crit.from(BalanceTransaction.class);
        Join<BalanceTransaction, Organisation> org = obtRoot.join(BalanceTransaction_.organisation);
        Join<BalanceTransaction, Item> item = obtRoot.join(BalanceTransaction_.item);
        Join<Item, AgreementInvoice> agreementInvoice = item.join(Item_.agreementInvoice);
        Join<Item, StatusTransaction> statusTransaction = item.join(Item_.statusHistory);

        List<Selection<?>> selectionList = new ArrayList<>();
        selectionList.add(obtRoot.get(BalanceTransaction_.id).alias("id"));
        selectionList.add(agreementInvoice.get(AgreementInvoice_.id).alias("agreementId"));
        selectionList.add(agreementInvoice.get(AgreementInvoice_.reference).alias("agreementReference"));
        selectionList.add(org.get(Organisation_.id).alias("organisationId"));
        selectionList.add(org.get(Organisation_.reference).alias("organisationReference"));
        selectionList.add(org.get(Organisation_.organisationType).alias("organisationType"));
        selectionList.add(obtRoot.get(BalanceTransaction_.endDate).alias("endDate"));
        selectionList.add(obtRoot.get(BalanceTransaction_.creationDate).alias("creationDate"));
        selectionList.add(obtRoot.get(BalanceTransaction_.effectiveDate).alias("effectiveDate"));
        selectionList.add(obtRoot.get(BalanceTransaction_.value).alias("value"));
        selectionList.add(obtRoot.get(BalanceTransaction_.balanceType).alias("balanceType"));
        selectionList.add(obtRoot.get(BalanceTransaction_.status).alias("status"));
        selectionList.add(item.get(Item_.identification).alias("itemIdentification"));
        selectionList.add(item.get(Item_.id).alias("itemId"));
        crit.multiselect(selectionList);

        Predicates predicates = Predicates.create().add(
                cb.equal(agreementInvoice.get(AgreementInvoice_.currency), rp.getLocalCurrency()));
        predicates.add(cb.equal(obtRoot.get(BalanceTransaction_.status), ItemStatusType.ACTIVE));
        predicates.add(cb.equal(obtRoot.get(BalanceTransaction_.balanceType), BalanceType.COLLECTION_BALANCE));
        predicates.add(
                cb.or(
                        cb.between(statusTransaction.get(StatusTransaction_.effectiveDate), rp.getStartDate(), rp.getEndDate()),
                        cb.and(
                                cb.lessThan(statusTransaction.get(StatusTransaction_.effectiveDate), rp.getStartDate()),
                                cb.or(
                                        cb.greaterThan(statusTransaction.get(StatusTransaction_.endDate), rp.getEndDate()),
                                        cb.isNull(statusTransaction.get(StatusTransaction_.endDate))
                                )
                        )
                )
        );
//        predicates.add(cb.equal(item.get(Item_.itemType), ItemType.INTERNAL_ACCOUNT));


        // .add( cb.or (effectivedate, cb.and ( 124, 125, cb.or (enddate statustransaction))
        //Inner queries are from disjunctions or conjunctions, disjunctions is an OR case
        // Conjunctions are AND
        // So below is : A OR (B AND C AND (D OR E))

        return ImmutableList.copyOf(new HashSet<>(getTransformedResultList(
                crit.multiselect(selectionList).where(predicates.get()),
                TupleToBeanTransformer.create(
                        ValueTransactionRow.class))));
    }
```
- Daily Dealer Reports in UI config is how to enable a report being shown up for a DEALER to be able to select, therefore it needs to be added in the Daily Dealer Reports map in ReportContext.xml

### Post-mortem
#### Why did it take longer than expected
- Lack of confidence from developer to challenge original estimate
- Didn't fully take in complexity from the start, overwhelmed with general gap
- Led to me not knowing how to break the task down
- Felt like I couldn't retain all of the ticket info in my head at all times
- Being stuck on report layouts
- Honestly major big hiccup was getting the layout displaying 
- Complexity increases with not fully defined AC
- Figuring out reports and how they work as a whole took longer than expected - still not 100% there
- Change figured out in the latter half where a lot of work needed undoing to try and break the normal style of reports and have multiple sections coming from one scripted dataset
- Business knowledge in domain lacking so recreation of scenarios not easy


### Notes from Post-Mortem
Feedback loop of why things happened as they did, trying to be blameless as possible and what slowed down, what wasn't understood. Process for future:
- Set up an hour or so long meeting with Rich, Chris, or Patrick about confusion points after reading
- Crash course on business context with BA for sure
- Spend half a day breaking into subtasks
- Look into Chris, Patrick or Rich's outlook calendars
- Raise just before hitting deadline that it won't be made, raise formally
- Communicate, *communicate*, **communicate**



#### Issues
- Started off trying to understand everything and got overwhelmed
- Didn’t challenge the estimate from the start
        - Didn’t feel comfortable making that challenge
        - Missed the warning sign there
        - Not enough contingency in it for the junior dev
- Got stuck multiple times, muddled through each
        - Not a big block, but small parts slowing it down
        - Possibly needed to reach out more from here
- When it was obvious that it would take longer, didn’t feel comfortable saying how much by
- Needed a more upfront go through
        - Cover everything that wasn’t clear
        - Look for similar previous work for ideas
        - Struggled with how the report layouts and scripted datasets work
                - Moving parts and how they connect together
                - how different reports with a similar layout can look vastly different
                - struggled to use legacy work as an example
                - hard to debug
- Approach wasn’t so clear with regards how to code the scripted datasets so went from using multiple, to just one, to multiple again and the back and forth lost time
        - Greatly increased the complexity
        - Wasn’t a scenario foreseen in the dev approach
- Not enough business context knowledge to make sense of a lot of it
        - E.G debits and credits being flipped for internal accounts
        - Lost time trying to fix things which weren’t an issue
- v5 sample documents
        - Caused confusion as that contained negative values that don’t fit with how v6 works
#### Mitigation: (checklist)
- Picking up the task
        - Read through
        - Call/Meeting with seniors to talk it through
                - Maybe the BAs too
                - Discuss how to break it down in to subtasks
        - Re-evaluate the estimate
- Split the ticket in to smaller subtasks
        - More admin
        - Easier to estimate the smaller chunks
        - More touch points to step back and re-evaluate if the bulk of the work it on track or not
        - Code reviewed earlier
- Book meetings in the calendar to make sure there is time for help to be got
        - Don’t worry about how busy we are
- Need to work on the approach to tickets generally so the steps from this are applied to everything going forwards
- Don’t just challenge the estimate at the start, challenge the moment it is going wrong, don’t wait till the end of the estimate
#### Things that went well & Lessons Learned
- Learned
        - To work more collaboratively with the team
        - Helped build communication with the team
        - Using the UI and using the DB as the source of truth
        - More about bean wiring
        - How to look through the legacy code better
        - To use the report layout knowledge gained from this to help others on the team
        - Better understanding of how the DAOs and Service layers work
        - Better understanding of report layouts and how they link to scripted data sets


## Breakdown of solution
### Translations added
- `translationManager.getMessage("invoice_date", rp.getOrganisationLocale(), null)).style(ReportGenStyleType.DEFAULT_HEADER)` added to ReportLayout to set a column heading to a translatable thing so that in Admin -> Translation Admin this can be changed and reflect a different value, however you have to refresh the message source for this to work

### Beans added and constructor injection
- Beans for the ReportService and all 4 ScriptedDataSets but with the switch to constructor injection changes `<property name="" ref=""/>` to `<constructor-arg ref=""/>" and these follow the ordering of the constructor parameters.
- Then in the respective classes instead of *Setter* methods with the `@Required` annotation, there are `private final ` variables that get set upon construction.

### DAO methods and use in service layer


### Sanity testing
- Wait for build to be released on jenkins
- Update medusa (our HoB) to the latest
- Wait for it to have updated
- Test



## Testing Multithreading
```
// mock dao query
// call service method
// assert cached result is one you mocked
```
- Cached service basically runs a `getCache()` at the start of *every* method, and then runs a check to see if the results are null or need populating i.e.
```
if (cache.getInterestDataResults() == null || cache.getInterestDataParams()
                .doesInterestDataNeedPopulating(rp.getStartDate(), rp.getEndDate(), itemType))
```
*(this is for interest obviously changes for transaction, charges, account details etc)*

The way this works is the cache is a way to store the results returned from the database in memory, so for huge datasets you hit the database much less. Most of the queries contain `item type, start and end date, currency` and to populate the cache we pass `null` as ownerId so that we get ALL ROWS instead of per dealer, because the cache wants to be organised into multimaps of `(ownerId, nonWholesaleDealerStatementRows)`, a good check might be checking that the ownerId exists in the rows we're returning from the mock.
