---
id: 79is3g8ias18k3ldsfd54r0
title: Fleet
desc: 'A file for knowledge on stuff to do with Fleet/Johto project'
updated: 1686920348580
created: 1686919376670
---
### Basics
The key aspects of Fleet that we're focusing on for now are going to be:

- Implementing changes to enable loan repayment schedules to be calculated using the Freehand API (this is another sopra product we'll be interacting with) and to capture the information needed from the user during loan creation to send to Freehand.
- Adding a new asset populator for CAP Gold, this is basically just adding a new way to populate details about an asset automatically from an identification key such as a VIN, which hits the CAP Gold
- Adding metrics + reporting for tracking CAP Gold usage

> Things of note: PPP, Asset Populators and Loan Creation

### Product, Profile, and Plans
[Confluence page](https://confluence.apak.com/live/pages/viewpage.action?spaceKey=WIKI&title=Product%2C+Profile+and+Plans)
These store all the information on how a *loan/item* is created in SFP-W/WFS.
