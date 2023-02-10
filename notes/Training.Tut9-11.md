---
id: sbp845l9og1c7wpen76izp1
title: Tut9-11
desc: ''
updated: 1675953980494
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

### 6-8 Missing bits
- **Spy** - wraps an existing instance and behaves the same but tracks interactions
- **Property & Constructor Injection** - Property injection is better for switching things at runtime, whereas constructor injection is better for enforcing dependencies.
- **`@Qualifier`** - Can be used to solve multiple beans of same type when using autowired, is the same as using the `@Resource` with the *name* parameter.
- **Sequences** - something that follows a pattern or a rule, in databases allows automatic generation of values such as check numbers. Good for generating unique key values.
- **Normalisation** - for elimination of redundancy of data, its for removing duplicates from tables and expanding to more tables, but too much normalisation causes performance issues. Library example, want a record for each book, but also probably want each time a book is taken out and returned, but these would go into other tables.
- **Optimistic and Pessimistic Locking** - optimistic locking is where you take note of a version number and check its unchanged before writing the record, whereas pessimistic locking is the locking of a record for exclusive use until finished with it
- **`@XmlID`** - another way of declaring something as an ID 
- **Criteria benefits** - isn't vulnerable to SQL injections as queries are dynamically generated, more scalable, changeable and readable that an HQL query, because of object oriented API, errors can be detected at compile time.
- **Criteria alias** - a way of renaming projections and making them more reusable
- **Aggregate functions** - calculate the final result using property values of all objects satisfying the given query criteria (*min, max, sum, avg, count*)
- **Detached Criteria** - a way of creating a query without the session, then you can reuse searches regardless of session.


#### Notes
Mocks extend a class and make an instance of the extension, whereas Spy wraps the instance and then by default uses normal methods but through stubbing can override.
Pessimistic locking is better in highly critical systems and maybe banking, then there is pessimistic plus which locks a whole table.
*Create alias* in hibernate creates a join and gives it a name for a join.
*CSRF - Cross site request forgery* - token given out 
XSS - cross site scripting, injecting malicious code that gets stored in the website
<br> Session management - last logged in, how long logged in, a way of identifying who you are