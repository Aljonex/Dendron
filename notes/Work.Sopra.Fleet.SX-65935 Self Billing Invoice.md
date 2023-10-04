---
id: qairy24kynu9im00k6gzdpz
title: SX-65935 Self Billing Invoice
desc: ''
updated: 1695821370006
created: 1695396721898
---
### Tickets
- [WMOD-1877](https://jira.apak.com/browse/WMOD-1877)
- [SX-65935](https://jira.apak.com/browse/SX-65935)
### PRs
- [WMOD-1869]()
- [SX-65935]()


### Steps for model
First I used the already made `InvoiceType` of `SELF_BILLED_INVOICE` and based my model changes off of that:
```java
SELF_BILLED_INVOICE(
        "self_billed_vat_invoice",
            TaxExtract.EXTRACT,
            InvoiceDirectionType.PAYABLE,
            PurchaseSalesType.PURCHASE), //Self Billed Invoices
SELF_BILLED_INVOICE_WITH_PARENT_FUNDED_FEES(
        "self_billed_vat_invoice_with_parent_funded_fees",
        TaxExtract.EXTRACT,
        InvoiceDirectionType.PAYABLE,
        PurchaseSalesType.PURCHASE),
```
Then `ctrl+f` for `SELF_BILLED_INVOICE` and added all of my new invoice to the relevant `isX()` methods and switch statements, also decided to use `self_billed_invoice_with_parent_fundeded_fees` instead of `fleet_sales_invoice` due to the design decision from GAP013. This was in method `InvoiceType.getDocumentTitleKey()`.

### Solution
Accidentally made changes regarding this to `CollectionCapitalInfoItemsGenerator` which caused some issues but quickly realised it was the wrong place to put the `SELF_BILLED_INVOICE_WITH_PARENT_FUNDED_FEES` InvoiceType.

- `BaseSystemX` - addition of new `InvoiceType` to beans for spring loading where needed, firstly in the `bureauInfoItemGenerators`with correct generator (for this is was `selfBilledInfoItemsGenerator` *different* to GAP013 SX-65934), also added a new document creator similar to ones already made, then added to the map of `newAgreementTaxFacilityConditionalDocumentCreator` linked to `landscapeInvoiceIdentifiersMap` which in turn was linked to the `portratInvoiceIdentifiersMap`
- `configurations.xml` - added the document creator created in BaseSystemX to a map of `documentCreators` selectable in config admin on the UI
- In relevant `Translations_en` added the new InvoiceTypes
- Added the new invoice type to generic switch statements and followed how the current `SELF_BILLED_INVOICE` was listed anyway, added it to `ValidInvoiceTypesForPlanDetails`