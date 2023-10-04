---
id: 1yq5533cgzb9we89a8hbplj
title: SX-65934 Upfront Invoice
desc: ''
updated: 1695889760907
created: 1694521897321
---
### PRs
- [WMOD-1869](https://bitbucket.apak.delivery/projects/BRK/repos/wfs-model/pull-requests/2231/overview) - [PR](https://bitbucket.apak.delivery/projects/BRK/repos/wfs-model/pull-requests/2231/overview)
- [SX-65934](https://bitbucket.apak.delivery/projects/WFS/repos/wfs/pull-requests/30213/overview) - [PR](https://bitbucket.apak.delivery/projects/WFS/repos/wfs/pull-requests/30213/overview)

[[SX-65935 Self Billing Invoice|Work.Sopra.Fleet.SX-65935 Self Billing Invoice]] 


Had forgotten to wire in the `COLLECTION_CAPITAL_WITH_PARENT_FUNDED_FEES` `InvoiceType` into the `BaseSystemX.xml`, went through this programatically and followed suit of how the `COLLECTION_CAPITAL_NEW` was laid out.

Was getting a `failed to load application context` error, this was down to the fact that in `BaseSystemX.xml` I'd copied the `COLLECTION_CAPITAL_NEW` bean from:
```xml
<bean id="newAgreementTaxDocumentCreator"
		class="com.apakgroup.systemx.process.paperwork.NewAgreementInvoiceDocumentCreator"
		parent="baseDocumentCreator">
		<property name="invoiceType" value="COLLECTION_CAPITAL_NEW" />
		<property name="invoiceCategory" value="TAX" />
		<property name="documentDcInclusionMap" ref="newAgreementTaxDocumentDcInclusionMap" />
		<property name="deletePendingInvoiceBusinessProcessTypes"
			ref="newAgreementDeletePendingInvoiceBusinessProcessTypes" />
		<property name="validSystemProductTypes">
			<set>
				<value>WHOLESALE</value>
			</set>
		</property>
	</bean>
```
But replaced the invoice type and `validSystemProductType` without then also updating the `util:map id="documentCreatorConfigurations"` in `configurations.xml` so to rectify this, I added an entry for my `InvoiceType` and added a value-ref to be called in `BaseSystemX` called `newAgreementFleetInvoiceDocumentCreator`.


### Sidenote
- Deposits are changing so this ticket will need revisiting when this comes about