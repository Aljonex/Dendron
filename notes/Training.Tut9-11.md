---
id: sbp845l9og1c7wpen76izp1
title: Tut9-11
desc: ''
updated: 1675247145083
created: 1675180096057
---
### Tutorial 9
- What is JSF?<br>
*JavaServer Faces* is a technology that includes a set of APIs for representing UI components and managing their state, handling events and input validation, defining page navigation, supporting internationalization and accessibility.
It also has *JavaServer Pages* custom tag library for expressing a JSF interface within a JSP page.<br>
Its a framework for building web applications and takes a component-centric approach for developing Java Web user interfaces.
- What is a Deployment Descriptor and what is it used for?<br>
JSF applications get packaged in a WAR file and it must conform to specific requirements to work across different containers.
It must have a *deployment descriptor* which is the `web.xml` which configures resources required by a web app.
- What are JSF's lifecycle phases?<br>
    1. Restore View - retrieves component tree for requested page from FacesContext instance if its already displayed, otherwise it creates a component tree and saves to the FacesContext instance.
    2. Apply Request Values - components in the tree retrieve their values from the request and cache them (store locally)
    3. Process Validations - all convertors and validators are executed on local values and if validation passes all phases executes normally otherwise JSF implementation adds error to FacesContext instance
    4. Update model values - bean properties updated using corresponding component local values
    5. Invoke application - business logic executed and string returned passed to navigation handler for navigation
    6. Render Response - response sent to browser
- What are basic JSF tags and attributes?<br>
`inputText inputSecret inputTextArea selectStuff outputStuff links` and for attributes `id binding rendered styleClass value valueChangListener converter validator (and messages for each)`
- What are composite components?<br>
Allow custom component definition, defined in their own `xhtml` folder, can define custom attributes and its implementation
- How does JSF navigation work?<br>
JSF provides an auto view page resolver mechanism called implicit navigation, you put the *view name* in the *action attribute* and JSF searches the correct view page automatically in deployed application. This can also be done in managed beans which are controllers, and will return strings of the name of the xhtml page.

### Tutorial 10
- OWASP top 10 web vulnerabilities and protection against them
![[Training.Security]]

### Tutorial 11
- What is WCAG and how it categorises<br>
*Web Content Accesibility Guidelines* international standard, it has 4 principles which are *perceivable, operable, understandable, robust* and 3 success criteria levels *A, AA, AAA*.
1. **A** is the basic level which is achievable with any website, should work reasonably well in text only mode, all text and videos should have text alternative for browsers that don't support and all links should be viewable.
2. **AA**, use html code correctly, style sheets to control layout and presentation, text based browsers ignore CSS but the page should still be readable.
3. **AAA**, this is for making it the most accessible for disabled people. Like having text included between links, the WAI recommends the use of navigation bars for clear navigation along with markers to show visitors where they are on the site.
Also creating a logical order for links and forms to allow a visitor using the tab key to tab down the page.
- Know key accessibility aspects mentioned in the tutorial<br>
    1. Input fields
    2. Buttons and Links
    3. Titles and headings
    4. Data tables