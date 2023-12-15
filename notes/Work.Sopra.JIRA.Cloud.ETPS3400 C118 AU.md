---
id: qme42xg1nrffaioeiik34b2
title: ETPS3400 C118 A
desc: ''
updated: 1700650708285
created: 1699529543870
---
[JIRA](https://jira.apak.com/browse/ETPS-3400)

```
Tenant:  c118-au
Customer:  c118
Region:  AU Melbourne
Subdomain:  hcau
Service Name: Hyundai Capital Australia
Client Email Identifier Pattern:  @hcs\.com${}
Allow non-matching email addresses: true (their current emails are @hcs.com and @au.hcs.com)
Internal Owner Name: Matt Wilcox
Internal Owner Team: Product Management
```

Asked James R how to progress on this, he gave me a link on [AWS regions](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html#concepts-available-regions) so I could understand why we had an `ap-southeast-2` and `ap-southeast-4`, these are both Aus regions (Sydney and Melbourne).
However, he advised me to pushback as there's no `wfs-production.git` repo under `APSE4P` (the group of production repos), which to make a Cognito pool as laid out in step 1 of [tenant definition](https://bitbucket.apak.delivery/projects/DOC/repos/design-and-process/browse/sfp-w/cloud-environments/environment-setup/tenant-setup.md) means its not possible.


### Repos needed
- `wfs-production`
- `cluster-config`


### Steps
- First step will be to get this prod repo up, then make changes to `cloud-deploy-template` under `profiles/tenant-setup.yaml`
```yaml
metadata:
  region:
    eu-west-1:
      prodrepourl: https://bitbucket.apak.delivery/scm/awsp/wfs-production.git
    us-east-2:
      prodrepourl: https://bitbucket.apak.delivery/scm/use2p/wfs-production.git
    ap-southeast-4:
      prodrepourl: https://bitbucket.apak.delivery/scm/apse4p/sfpw-production.git
```

- Next step following this [tenant-setup](https://bitbucket.apak.delivery/projects/DOC/repos/design-and-process/browse/sfp-w/cloud-environments/environment-setup/tenant-setup.md) was a *Cognito Pool setup*, using the cloud-deploy-template.
(The repo this is in is `prod-infrastructure-tf`)
However, when the PR was up Chris Wall mentioned we need mapping:<br>
In `main.tf`:
```yaml
module "c118_au_user_pool" {
    source         = "./cognito_pool_c118_au"
    aws_region     = var.aws_region
    stage          = var.stage
    owner          = "Matt Wilcox"
    owner_function = "Product Management"
    customer       = "c118"
    external_id     = module.base_security.sns_access_external_id
    sns_access_role = module.base_security.sns_access_role_arn
    map_tag_server_id = data.vault_generic_secret.map_tags.data["serverId"]
}
```
Then in the cognito folder for the env:
```yaml
locals {
  name = "c118-au"
  tags = {
    Service       = "Authentication"
    Role          = "Authentication"
    Owner         = var.owner
    OwnerFunction = var.owner_function
    Customer      = "C118"
    Stage         = var.stage
    map-migrated  = var.map_tag_server_id
  }
}
```
And this needs to be added to `variables.tf`
```yaml
variable "map_tag_server_id" {
  description = "The value of serverId used for the Amazon Migration Acceleration Program"
  type        = string
}
```

- Tenant definition, the ticket wants me to match against identifiers `@au.hcs.com` and `@hcs.com` so went on to [Regex101](https://regex101.com/) and tested both `@(:?au.)hcs.com` and `@(:?au\.)hcs\.com` and both have matches for those patterns. Had to find the `c114` env in the APAK VELA service registry tenants repo to see the example of additions to the `navigation` and `platform-credentials-provider`
```yaml
services:
  - feedback
  - id: navigation
    params:
      - key: platform/regions
        value:
          - '*' #Has to be all regions for now due to navigation being used in a global way in the wrapper
  ...
  - id: platform-credentials-provider
    params:
      - key: platform/regions
        value:
          - ap-southeast-4
```
### Cognito
- Merge the cognito pool before tenant definition and monitor in `CodeBuild` in AWS production "Software Engineer" management console, then under build history, find it based on when you started it, it will have some link with `in-<repoName>` so for the cognito pool mine was `in-prod-infrastructure-tf`.
### Tenant Definition
- Then merge the tenant definition. It'll first do a `GitPullCodeBuild` like for the cognito.
### Kube
- Next is Kubernetes setup, first I had to update the `cloud-deploy-template` so I put a quick fix in adding the [newer standards](https://bitbucket.apak.delivery/projects/SAN/repos/cloud-deploy-template/pull-requests/105/overview) which involved in the `@@TENANT@@.yml` changing the `identifierPattern: "@@IDENTIFIERPATTERN@@"` to single quotes as yaml doesn't parse values inside double quotes, not sure why this was like this in the first place.<br>
Then adding `map_tag_server_id` to the `template/tenant-setup/cognito-setup/prod_inf_tf` so in future all cognito pools have `map_tag_server_id = data.vault_generic_secret.map_tags.data["serverId"]` in the tenant user pool. Then linking this to the `variables.tf` and making a slight change to the `main.tf` where tags are stored in local variables, and the map is listed there too.
- Before completing this step I had to add the Asia-Pacific deployment template of kube setup, very similar to the ones above however it wants to put properties into the `cluster-config/argocd/projects/wfs-production.yaml`, now I need to understand the design decision for this, its an easy link from the template and there's already the repo sfpw-production, I think it would be best to delete and remake that repo and call it `wfs-production` for consistency, otherwise migrate the rest, but also policies are called `wfs-production*` which means potential migration for those too?
- Kept failing on a file not found error, when going to this the root of this error was from `git_task_executor.py` line 26 where it does git checkout, as I'd tried this without the file I already had the branch off of master and it meant it was just re-checking out that branch, had to delete and rerun tool.
- Couldn't push to a repo after pulling with `http` and recurse submodules, turns out I just needed to make new http access token with write permission