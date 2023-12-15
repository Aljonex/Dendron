---
id: 443pb7j75wr9jo9dg27d0ww
title: WebUI
desc: ''
updated: 1702570377376
created: 1686730623670
---
## ETPS-1785 More actions scrolls
Originally thought it was an error to do with JSF, and found the `LoanAccountSearch.xhtml` page for the page I was locally recreating with `DC selected -> Loan Management -> Capital Loans -> Loan Search -> Search Views = Loans & Item types = Invoice`. 
Also, found out that the error doesn't only occur on `More Actions` being clicked but also `Settings` (less noticeable as unlikely to click this when scrolled down).
Found both of these on `table.css` under names of `.ApakTableSettingsButton` and `TableExtraOptions`, then found both of these on `ApakTableRendererOld` and thought it was something to do with that.

However, spoke to Jordan Hellier (JSF and UI wizard as was a major part of the UI reskin), and after making use of the Dev Tools we saw that on clicks of these buttons (under network) no request was made at all, which when clicking links happens. Meaning it was nothing to do with JSF or the page itself, leaving only JavaScript as the culprit as that is what otherwise controls page logic as html and css is for layout and styling.
- Grunt
    - At the root of WFS-UI folder and basically compiles all `.js` files into a concatenation that makes up the logic of javascript on the page.
- Instead of reloading Tomcat
    - Can drag `js` or otherwise files into the `target` under the same folder
- WFS-UI
    - Once it was found out to be the JavaScript instead of the JSF or java it meant that we wanted to load the `wfs` and `jsquery` non-minified files, under `WebRoot -> scripts` but to make changes to these [this page must be followed](https://confluence.apak.com/live/pages/viewpage.action?spaceKey=WIKI&title=How+to+modify+WFS+JavaScript).
    - To get non-minified, there is a thing in `mainLayout.xhtml`
    ```html
    <c:choose>
        <c:when test="#{uiHelper.developmentMode}">
            <ui:include id="scriptInclude" src="devScriptsAndSheets.xhtml"/>
        </c:when>
        <c:otherwise>
            <ui:include id="scriptInclude" src="cdnScriptsAndSheets.xhtml"/>
        </c:otherwise>
    </c:choose>
    ```
    And to make a change to this to allow development mode we must either add in `local.conf` `env.development=true` or add a breakpoint in `UIHelper.developmentMode` and `right-click -> more` and instead of suspending, on breakpoint `Evaluate and log -> this.developmentMode=true`
- [Developer Tools in Chrome](https://developer.chrome.com/docs/devtools/)
    - Network 
        - Shows when and where requests are made, in the example of this I could see that clicking a button made no requests
    - Console 
        - Can add things like `document.addEventListener('scroll', ()=> console.log('scrolled'))`, on page refresh all of these get cleared
    - Elements
        - `ctrl+shift+c` = select an element to inspect (top left button)


`ApakTableRendererOld`, my gut instinct was right! These 2 elements are the only ones still in the old renderer, Jordan noticed there may be a hyperlink change and upon clicking there was a `#` added to the end of the link in `href` and through a quick google search 

    As defined in the HTML specification, you can use href="#top" or href="#" to link to the top of the current page. Yes, it is easy to scroll to the top by using the html's <a> tag.

The lines that added a `#` to the Href in `ApakTableRendererOld` were `writer.writeAttribute(ApakTableRendererUtils.HREF, "#", null);` and as I thought there were exactly 2 instances of this, both could be removed but this is only for the more actions.

### After merging
Had forgot to check for tests, found that it failed `ApakTableRendererUTest` on 3 methods `testEncodeEnd` `testEncodeBeginWithSubheaderInsteadofHeader` and `testEncodeBegin`, this was because there are 3 text files namely `encodeEndCorrectOutput.txt encodeBeginCorrectOutput.txt` and `encodeBeginWithSubheaderInsteadOfHeaderCorrectOutput.txt` and these all had checks against either `moreActions` or `settings` that was verifying there was `href="#"` which was my change, had to remove this bit and then push and create PR, because it was too quick I couldn't `build with regression tests` so I had to go to [Jenkins Regression Tests](https://jenkins.apak.delivery/job/WFS%20Regression%20Tests/) and `Scan multibranch Pipeline now` to find new branch, then could go back to my PR and test.


## Unnamed ticket, issue with Standard Bank with Max
Some mandatory fields weren't showing as being editable in Dealing Company level `Administration -> Organisations -> Clients`, for Standard Bank which runs on `8.47.388.15` but in Honda this worked (also `8.47.388.15`), so went to this page and found the relevant xhtml.
`adminOrgDealer.xhtml` and from there found that for General Details I needed to be on
```java
<apak:apakSubTabSet selectedTabId="#{adminOrgDealer.selectedSubTabId}" id="generalSubTabs">
                        <apak:apakTab id="mainDetailsTab" title="#{sysxmessage.main_details}">
                            <ui:include src="/inc/organisation/tab_adminOrgGeneralMainDetails.xhtml">
                                <ui:param name="adminOrg" value="#{adminOrgDealer}"/>
                                <ui:param name="entity" value="#{adminOrgDealer.organisation}"/>
                            </ui:include>
```
`tab_adminOrgGeneralMainDetails.xhtml`, also found the controller through the `BaseUI.xml` and under `adminOrg`
```xml
<bean id="adminOrgDealer"
		class="com.apakgroup.systemx.ui.organisation.dealer.DealerAdminController"
		scope="conversation" parent="anonymisableOrganisationAdminController">
    <property name="uccService" ref="uccService" />
    <property name="pmsiService" ref="pmsiService" />
    <property name="uccController" ref="uccAdmin" />
    <property name="pmsiController" ref="pmsiAdmin" />
    <property name="insuranceLimitDetailsController" ref="insuranceLimitDetails" />
    <property name="insuranceService" ref="insuranceService" />
    <property name="displayResolverFactory" ref="dealerSelectionDisplayResolverFactory" />
    <property name="permitOverrideAutoPaymentAllocation" value="${permitDealerOverrideAutomatedPaymentAllocation}" />
</bean>
```
From here I knew the 4 fields that weren't showing were *VAT number, Company Name, Trading Name, and Company Reg Number*, and for these there is a method for disabling called `#{adminOrg.isGeneralDetailsReadOnly('<variable>')}`

So I found the method:
```java
public boolean isGeneralDetailsReadOnly(String fieldName) {
        return isReadOnly() || isFieldReadOnly(WfsFieldEntity.DEALER, fieldName)
                || isGeneralSettingsReadOnly();
}
```
And also noticed that in the xhtml there were 6 instances of this being called, but both others were also readOnly, however they showed as used a `selectItems`, now in this method there are 3 `OR` statements.
`isReadOnly()` comes from a bean injection and due to other things shown on the page as editable also check this, I knew it was false, `isFieldReadOnly()` was hardcoded and I guessed this was false.
This left `isGeneralSettingsReadOnly()` which checked the permissions manager, Max assured me at PFC level he had these permissions, but I got him to double check and he didn't and VOILA it was fixed.
```java
public boolean isGeneralSettingsReadOnly() {
        return isReadOnly() || !(permissionsManager.hasAllPermissionRights(UserRight.EDIT_DEALERS,
                UserRight.EDIT_DEALERS_GENERAL_SETTING));
    }
```


### My `local.conf` setup
```
wfs {
     core {
     env.development=true
     logLevel = "DEBUG"

        # Postgres settings
        application.database.profile = "postgres"
        uiReskin.availableStylingOptions = ["ORIGINAL", "MATERIAL"]
        application.jdbc.postgres {
          username=SYSX_USER
          url="jdbc:postgresql://localhost:5432/"
          schema {
            partitions = [
              {
                # Default username comes from postgres profile (above)
                partition = "1"
              }
            ]
            defaultPartition="1"
          }
        }
        migration.flyway {
          locations = "db/migration/postgresql, db/migration/common"
          nonPartitionLocations = "db/common_data/migration/postgresql, db/common_data/migration/common"
          createcommonhibernatesequence = false
          #auto = false
        }
#         application.hibernate.show_sql=true
     	#translations.enabled = false
         application {
             jdbc {
                oracle {
                	url="jdbc:oracle:thin:@(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=localhost)(PORT=1521))(CONNECT_DATA = (service_name =xepdb1)))"
                  	schema {
		                entitySpaceMultiplier=1
		                partitions=[
		                    {
		                        partition="1"
		                    }
		                ]
		                defaultPartition="1"
		            }

                 }
                 global.url = ${application.jdbc.oracle.url}
                 //profile=oracle
             }
         }
         oauth.custom.users = [                {
                clientId="PasswordUser"
                secret="{noop}TestSecret1"
                grantTypes="password,refresh_token"
                authorities="SCOPE_wfsv6.io/rest,ROLE_ACCESS_REST_API"
                accessTokenValidity = 600000
                refreshTokenValidity = 600000
                scopes="wfsv6.io/rest"
            }
        ]
     }
}
```
This was for the addition of the UI reskin, also have to run `yarn install` then `yarn build` in target WFS-UI folder.
`yarn install` gets grunt, webpack and a few other dependencies, `yarn build` builds it


### Brian's tips
- `Alt+F1` (Intellij) for finding location of file
- Click file (bundled one) and `F5` which refactors and copies and move to the `target > exploded war > css > brandings > md` after `yarn webpack` and refresh UI
- Also `Ctrl + shift + C` (in dev tools in chrome [`F12`]) lets you manually inspect and click element to find
- Expand and can double click text to edit or drag out and see styles also to add new styling rule, then you can copy and paste
```css

tbody#mainForm\:alerts\:tbody_element {
    text-wrap: pretty;
}
```
i.e. I pasted this into the alertStyling.css


### Toggle Styling on cloud env
- Enable UserRight toggle UI styling on the departments, save it log out and reenter