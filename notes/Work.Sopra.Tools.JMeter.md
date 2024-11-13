---
id: g0ywhnbgsjfub6u0vruf9s9
title: JMete
desc: ''
updated: 1731487681440
created: 1731420744744
---
## JMeter Loan state machine
[JMeter Loan Machine](https://bitbucket.apak.delivery/projects/WFSPERF/repos/jmeter-loan-state-machine/browse)
- Pull the repo, but change from master branch to something like the ETPS 5580 847 fix branch
- Change your terminals java version to 1.8 and set the path with commands like
```bash
export JAVA_HOME="/C/Program Files/Java/jdk1.8.0_202"
export PATH=$JAVA_HOME/bin:$PATH

mvn clean verify -Dclient.code=I1 -DskipTests=true
```
Run with tests being skipped as there are 2 tests that fail on timeouts.

### Issues
After all of this, one test failure, due to there being one loan which has a **non-numerical** reference, so used an SQL query to solve this.
```SQL
SELECT * FROM item where identification= 'FEEDLOADNEWLOAN11'

UPDATE item SET reference = '100237' WHERE identification='FEEDLOADNEWLOAN11' AND itemtype='INVOICE'
```
Ran the first to find it, then the second to update it.
That **OR** use `modelWorldLiteRunner` as this just does orgs, creditlines and plans, no loans.

### My Specific problem set
I wanted to be able to create loans across different plans that some had:
- A CAP code with the loan attracting tax
- A CAP code with loan not attracting tax
- Loans attacting and not attracting tax
- Having prior interest on their asset event types
- facility type changes 

And more, so had to speak to Tim, with notes from the README.md can add a base level activity profile and overwrite the flag in the arg when running the maven command with `"-Dprofile.activityProfileFileName=exampleActivityFileFilename"` can also use similar with `profile.userPropertiesFile`

To look into how to break it out there are A LOT of variables you can set, most are using the `NEW_AGREEMENT` process but Tim said there's a process `MIGRATE_NEW_AGREEMENT` which allows you to set absolutely everything.
To look into the fields you can set might be worth looking at `feedAgreement.xsd` or like `newAgreementHandler.xml` and its relevant `.xsd` also Tim has done a lot of these style things in the c18 user.

- Because I had loans with CAP codes attached to them (setting up the CAP/HPI database locally) this caused issues