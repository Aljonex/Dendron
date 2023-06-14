---
id: 443pb7j75wr9jo9dg27d0ww
title: WebUI
desc: ''
updated: 1686740078865
created: 1686730623670
---
### ETPS-1785 More actions scrolls
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