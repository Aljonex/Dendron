---
id: zuapj15ybgwvqjq4qmn0qt2
title: Menu Updates
desc: ''
updated: 1679392849668
created: 1679392456253
---
### Removing funding request from menu
Find the level you need to be in to remove, for this ticket I had to be at either Dealing Company or Dealer level.
- Toggle unsaved changes and then look at the hyperlink to find the page name
- That page name will be the name of the `xhtml` file that you should remove
- Find the relevant page controller
- Find the section in the *enum* `PageIdentifier.java` to be removed also
- `ctrl+click` things like the identifier to find where in the project things lie
- Look in the `BaseUI.xml` to find beans to remove also, potentially the `SystemX.xml`