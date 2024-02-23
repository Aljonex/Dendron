---
id: lgt8ev3kk33rw4dua3kd6vo
title: SX-66790 Bulk amend
desc: 'For the ticket of bulk amend not displaying on organisation plan overrides'
updated: 1704190008403
created: 1702905947715
---
### 8.47.388 Local Recreation
- First enable Business rules in `Finance Product` level of `VALIDATE_MAX_GROSS_COST` and `VALIDATE_MIN_GROSS_COST` for `ONLINE, FEED, FEED_LOAD` with max cost of £350k and min of £2,000
- This is for c100, Haydock, check against their rules
- `MAX_GROSS` was reprocess always whereas `MIN_GROSS` was reprocess never

### Codebase
- From screenshots HAYD-523 want to multiselect values in Organisation plan overrides and have button available
- From codebase this comes from `OrganisationPlanOverrideDetailsHelper.java#isShowBulkAmendButton`
```java
/**
     * @param selectedRows
     * @param showApplyButton
     * @param showCancelButton
     * @return boolean This method contains the logic to render the Bulk Amend button on the
     *         Override Plan Screen For each selected Rows Bulk Amend button should appear if
     *         PropertyType and Parameter Reference for each selected Rows are same. Also the number
     *         of selected Rows should be greater than 1
     */
    public boolean isShowBulkAmendButton(List<OrganisationPlanOverrideRow> selectedRows, boolean showApplyButton,
            boolean showCancelButton) {

        return !showApplyButton && !showCancelButton && selectedRows.size() > 1
                && selectedRows.stream()
                        .map(row -> new Pair<PropertyType, String>(row.getPropertyType(), row.getParameterReference()))
                        .distinct().count() == 1L;

    }
```
So we can see its predicated on the property type and parameter references being shared by the organisation plan overrides. In the original ticket the property type is `Business Rule Parameter` and param reference is `Max Gross Cost`.
- Can check UserRights available in `Departments` and the relevant department, it seems that PFCDEPT has
```java
public boolean isShowBulkAmendMenu() {
        return permissionsManager.hasPermissionRight(UserRight.EDIT_ORGANISATION_PLAN_OVERRIDES)
                && permissionsManager.hasPermissionRight(UserRight.VIEW_ORGANISATION_PLAN_OVERRIDES);
    }
```
Both of these available 
- Debug with Tim confused him too, he took steps through most files and offered that since we know create is working, play with the xhtml file to try and force the bulk amend button to show
    - He said "poke it in different ways"
- After various poking in different ways, landed on putting logic of xhtml for bulk amend in button I knew worked (`create`), turned out all logic worked as intended:
```html
<f:facet name="bulkAmendButton">
    <apak:commandButton id="bulkAmendButton"
        rendered="#{organisationPlanOverrideSelection.showBulkAmendButton}"
        styleClass="button" action="#{organisationPlanOverrideSelection.bulk}"
        value="#{sysxmessage.BulkAmendValue}"/>
</f:facet>
```
Landed on it must be something to do with it being an `apak:commandButton` in `apak:buttonContainer`, found another custom button on a different page and traced back to `ButtonContainer.java`
```java
public static final List<ButtonType> primaryButtonPriorityOrder = Collections.unmodifiableList(
        Arrays.asList(
            ...,
            ButtonType.BULK
        )

public enum ButtonType {
        ...,

        BULK("bulkAmendButton");

        String facetName;

        ButtonType(String facetName) {
            this.facetName = facetName;
        }

        public String getFacetName() {
            return facetName;
        }
    }
```
- Also translations weren't working, first fix was to change the value to `bulk_amend_value` as this will auto resolve through `TranslationManagerImpl.java` but as there was a translation available this didn't seem like the best fix, looking at the [confluence training page on translations](https://confluence.apak.com/live/pages/viewpage.action?pageId=12687239) after looking at many, realised I may need to wire the translation manager into the `BaseOrganisationPlanOverrideSelectionController.java`

### Interesting points
- There are files `PlanOverridesWidget.java` and `widget_plan_overrides.xhtml` which seem to also have everything but Bulk Amend, may be worth looking into these if another issue along these lines arises.
```html
<h:commandButton id="bulkAmendButton" rendered="#{planOverridesWidget.showBulkAmendButton}"
                    styleClass="button" action="#{planOverridesWidget.viewPlanOverride}"
                    value="#{sysxmessage.view}">
    <f:ajax listener="#{widgetPanel.selectWidget(widget)}" render="#{cc.clientId}:widgetPopupHolder"
            onevent="function(){decorateDateTimePickers()}"/>
</h:commandButton>
```
```java
public boolean isShowBulkAmendButton() {
    return permissionsManager.hasPermissionRight(UserRight.EDIT_ORGANISATION_PLAN_OVERRIDES)
            && permissionsManager.hasPermissionRight(UserRight.VIEW_ORGANISATION_PLAN_OVERRIDES);
}
```