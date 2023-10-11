---
id: d846bmi6oax36k17qxa0no3
title: SX-65466 Bug Fix
desc: ''
updated: 1697013534754
created: 1696414259580
---
Bug fix for GAP004.1

## First part - Permissions
The bug was to do with inability to view specific pages when in a user without specific permissions.
The change had been to do with updating `UserRight` and `PageIdentifier` java files so that a user without
- Create Agreement
- Create Quick Loan
- Create Loan from approval
- Create loan from third party transer in
- Edit referral suspense queue
- View Fleet Stock Search
<br>but **with**
- Create Fleet Agreement
- View Skip Button
- Add Fee to new loan
- Remove Fee from new loan 
<br> *permissions*

The change simply involved adding certain `UserRight` enums to `PageIdentifier` pages, a brief mistake I made was simply adding all `UserRight` permissions, including `VIEW_SKIP_BUTTON`, this is incorrect as we don't want **EVERY** user with `SKIP` permission to have access to this page, but we do want users with `CREATE_FLEET_AGREEMENT` permission to have access to this page.

## Second part - Available business rules
This was to do with BusinessRules not being available in **Administration > Plan > Finance Product** and the FLEET product, under the Business Rules section in the Fleet Business rules pre-setup. I did some digging around and found some seemingly useful files:
- `WfsRuleName` - with enums of WfsRuleNames with their `WfsRuleType`
-  `businessRulesDetails.xhtml` which pointed to the `BusinessRulesDetailsController` which had `getBusinessProcessTypeSelectItems()` which is what displays the available rules on the page

From here I realised that these rules would most likely be wired in via spring beans so looked in the `BaseSystemX.xml` and this is where they were contained, it was then a means of adding `<value>FLEET</value>` to the `validSystemProductTypes` property.

I was unsure about some of these, Kyran asked if I'd searched directly for them and then worked backwards from the translations to the file.
<br>i.e. `ctrl+shift+f` for `Validate Organisation Credit Status` showed the master translation points to `VALIDATE_ORGANISATION_CREDIT_STATUS` which is a `WfsRuleName`, to remove the crap can search for `WfsRuleName.VALIDATE_ORGANISATION_CREDIT_STATUS` and this shows a method in the relevant file 
```java
@Override
    public RuleName getName() {
        return WfsRuleName.VALIDATE_ORGANISATION_CREDIT_STATUS;
    }
```
Then if I search `BaseSystemX.xml` for that filename (`OrganisationCreditStatus`), I can find the rule, and add `FLEET` to `validSystemProductTypes`.
```xml
<!-- GlobalBusinessRule - Check Credit Status of an organisation -->
	<bean id="organisationCreditStatus"
		class="com.apakgroup.systemx.controllers.alerts.globalrules.OrganisationCreditStatus">
		<property name="reference" value="OrganisationCreditStatus" />
		<property name="validItemTypes">
			<set>
				...
			</set>
		</property>
		<property name="validBusinessProcessTypes">
			<set>
				...
			</set>
		</property>
		<property name="validSystemProductTypes">
			<set>
				<value>WHOLESALE</value>
				...
                <value>FLEET</value>
			</set>
		</property>
		<property name="itemService" ref="itemService" />
		<property name="translationManager" ref="translationManager" />
	</bean>
```