---
id: pzyjpus1zcyr314k42jp218
title: SX-66752-experian-populator
desc: ''
updated: 1701771408596
created: 1701770594646
---
Adding an experian asset populator to `c100-test-sandbox2`

### Steps
- Check [Cloud environments](https://confluence.apak.com/live/pages/viewpage.action?spaceKey=WIKI&title=Cloud+Environments) and there is a link for *Enabling Experian on an environment*
- Do PRs, when able to merge into `AWS EKS Non-production eu-west-1` for the Haydock machine then sync the `c100-test-applications` and `resources`
- Aishwarya said problem still occurring
- Patrick said he search `.xml` for "**experian**" and came across `experianAssetPopulator` and `experianOnlyAssetPopulator` and saw how they were wired and what underlying services they make use of
    - Can also check `asp-configuration.xml`, that's wirings for what we see in the U.I when we select different available asset populators in `config admin`
In `CAPAssetPopulators.xml` and `valuationApplicationContext.xml`
```xml

(CAPAssetPopulators)
<bean id="experianAssetPopulator" parent="baseCapAssetPopulator"
		class="com.apakgroup.systemx.controllers.asset.population.CDCCapAssetPopulator">
...
<bean id="baseCapAssetPopulator"        
                      class="com.apakgroup.systemx.controllers.asset.population.CDCCapAssetPopulator">
	    <property name="lookupTypeResolver" ref="capLookupTypeResolver" />
		<property name="featureFlaggingService" ref="featureFlaggingService" />
	    <property name="costBreakdownCalculator" ref="costBreakdownCalculator" />
	    <property name="registrationPlateLookup" ref="registrationPlateLookup" />
	    <property name="valuationService" ref="capLookup" />
	    <property name="lookupService" ref="capLookup" />
	    <property name="enquiryLogService" ref="enquiryLogService" />
	    <property name="priceListName" value="CAP" />           
	    <property name="supportedAssetTypes">
	        <list>
	                <value>CAR</value>
	                <value>BIKE</value>
	                <value>LCV</value>
	        </list>
	    </property>
	    <property name="supportedCategories">
	        <list>
	                <value>NEW</value>
	                <value>USED</value>
	                <value>DEMO</value>
	        </list>
	    </property>     
	    <property name="populateColourFromLookup" value="${cdcassetpopulator.populatecolourfromlookup}" />  
	    <property name="itemService" ref="itemService" />
	    <property name="systemDateSupplier" ref="systemDateSupplier" />
		<property name="itemCostCalculatorOverrideResolver" ref="itemCostCalculatorOverrideResolver" />
		<property name="enquiryLogDeserialiser" ref="enquiryLogDeserialiser"/>
      </bean>	

(valuationApplicationContext)
<bean id="experianOnlyAssetPopulator" parent="baseDecodeOnlyPopulator"
		class="com.apakgroup.systemx.controllers.asset.population.CDCOnlyAssetPopulator">
```
- Aishwarya (checked with her) and Haydock just use experianAssetPopulator and not experianOnly
- As it requires CAP, it also required [Enabling CAP Pricelist Database on an Environment](https://confluence.apak.com/live/display/WIKI/Enabling+CAP+Pricelist+DB+on+an+environment) then when merged re-sync the application and resources on Argo. [PR](https://bitbucket.apak.delivery/projects/EUW1NP/repos/c100/pull-requests/80/overview)
