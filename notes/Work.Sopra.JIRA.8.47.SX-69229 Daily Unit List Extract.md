---
id: e6duebuw2kzvnyf6tj4186y
title: SX-69229 Daily Unit List Extra
desc: ''
updated: 1727466456081
created: 1725382810913
---
### Basics
- For c18 custom so added to the c18-custom.xml
- Also had to add an entry in the map merger (do KS on `SpringPostFixer`)

```xml
<bean id="c18MapMerger" class="com.apakgroup.wfs.base.util.SpringPostFixer">
		<property name="collectionMerges">
			<list>
				<!-- <value>dcsHavingSelfBillings, c18InList</value> -->
				<value>eodPaymentInterfaceConfigurations, c18EodPaymentInterfaceConfigurations</value>
				<value>sepaFilenameResolvers, c18SepaFilenameResolvers</value>
				<value>allNonTransactionalEODProcesses, c18NonTransactionalEODProcesses</value>
			</list>
		</property>
	</bean>

	<bean id="dailyUnitListExtractEndOfDayProcess" class="com.apakgroup.wfs.customer.c18.batch.DailyUnitListExtractEndOfDayProcess" parent="dayEndProcess">
		<constructor-arg name="dayendService" ref="dayendService" />
		<constructor-arg name="organisationService" ref="organisationService" />
		<constructor-arg name="fileProductionManager" ref="${fileProductionManagerImpl}" />
		<constructor-arg name="outputDirectory" value="${configuration.root.output.dir}" />
	</bean>

	<util:map id="c18NonTransactionalEODProcesses">
		<entry key="dailyUnitListExtractEndOfDayProcess" value-ref="dailyUnitListExtractEndOfDayProcess" />
	</util:map>
```

- Otherwise followed examples of other extract files, looked in both bitbucket at custom (and most recent) like `C14FundingExtract` and Gerard's `CRSExtractEndOfDayProcess` to figure mine out
- Still need to break inner class into DTO class and then write unit tests to make sure a file is generating as intended
- Next step is adding DAO query and then service for data manipulation which will then be passed to my `DailyUnitListExtractEndOfDayProcess`.

### Testing
- Got a dump file from Cass, had to rename the file to `CAT.dmp` as the casing mattered - saved in downloads
- Moved to wsl with `\\wsl$` in the file explorer
- Moved to `local-dev-compose/oracle-18-xe/scripts` in a folder called `dmp_dir/` 
- Followed [this setup](https://sf-wiki.atlassian.net/wiki/spaces/WIKI/pages/41223462)
- Could then see it in `sqldeveloper`
- downloaded wiremock standalone and the [repo](https://bitbucket.apak.delivery/projects/WM/repos/wiremocks/browse)
- Put the standalone in the repo and ran `java -jar wiremock-standalone-3.9.1.jar --port 8081 --root-dir hpi --global-response-templating --verbose --print-all-network-traffic`