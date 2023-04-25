---
id: lidrzlw1ndk6p4mio0ecg40
title: Business Rules
desc: ''
updated: 1681923029187
created: 1681918813163
---
### SX-65058 Remove CreditLineIsValidRule
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