---
id: hwjmvidpp5usxx0eb2nb03u
title: Yarn
desc: ''
updated: 1702472177627
created: 1702376677233
---
Setting up NPM and Yarn
Set Nexus as your npm registry:
In your home directory (e.g. /c/Users/drodgers) create a file named .npmrc (don’t forget the leading dot).
Go to https://vault.apak.delivery/ui/
Select the OIDC login method and type “read-only” in the role box, then click the sign in button. This will sign you into Vault using Apak SSO.
In the Secrets tab, navigate to secret/prod/apak/services/nexus/npm/read-only. You should see two secrets named “username” and “password”. These can be copied to clipboard using the copy icon – this will be needed for the next step.
Resolve the basic authentication credentials for Nexus by running the following command in Bash, substituting values copied from Vault into the specified curly braces (note that the brackets should not be in the final value):

printf "<username>:<password>" | base64

In the .npmrc file you created earlier, type the following, substituting the result from the step above into the specified curly braces:

registry=https://nexus.apak.delivery/repository/npm
always-auth=true
_auth=<result of step 1e>

Download Node.js 14.x https://nodejs.org/dist/latest-v14.x/
Install Yarn https://classic.yarnpkg.com/en/docs/install#windows-stable


### Pain points
`bash: yarn command not found` - after doing `npm install -g yarn` and it working (this predicated on me having to add registry scope to my `.npmrc` file)
[registry scope](file:///C:/Program%20Files/nodejs/node_modules/npm/docs/output/configuring-npm/npmrc.html)

Had to add a new variable to system variables - [PATH](https://dev.to/monierate/yarn-command-not-found-on-windows-10-36pe)

- then run `yarn install` in WFS-UI