---
id: y72ctbz0tvytgwxu6pseo9g
title: Terraform
desc: ''
updated: 1686036349225
created: 1685517303532
---
### ETPS-2470 Stand up test machine for resilience tests
First had to pull the `prod-infrastructure-tf` as thought I was doing this for Ireland then found out that we were meant to be doing it for US-East-2 (Ohio) so instead had to pull `prod-us-east-2` whilst following Gerald's doc.
All edits on `.tf` files (terraform) for the database, a new module was needed in `main.tf` for the EKS worker a new module was needed in `eks.tf` and for the new EFS volume it was needed in the `efs.tf` file.

So first time push `atlantis` will auto build but after that must comment `atlantis plan` to call the webhook to rerun the terraform build.
To monitor the status you must go to https://apak.awsapps.com/start#/

Download `cluster-config` repo for correct region, in this case 'US-East-2', and make changes following document, new yaml.

When setting up temporary roles, in `master-infrastructure-tf -> users_bristol.tf` you can see your string module for the user being added for your `module <testing>`

```js
module "resiliency_testing" {
  source = "./role"

  identity_store_arn = local.master_identity_store_arn
  name = "Resiliency-Testing"
}

resource "aws_ssoadmin_account_assignment" "<firstname>_<lastname>_resiliency_testing" {
  instance_arn = local.master_identity_store_arn
  permission_set_arn = module.resiliency_testing.role_permission_set_arn

  principal_id = module.<email_first_name>_< email_last_name>__soprabanking_com.id
  principal_type = "USER"

  target_id = aws_organizations_account.apak_production.id
  target_type = "AWS_ACCOUNT"
}
```

If database is populated with wrong data you want to go into the sandbox and change (for this case) `resiliency-test.yaml` file under wfs-conf `conf.wfs.core.modelworld.loaders` then make a new pr.

If wrong data then its under `Sandbox resources` in argo and you want to delete the dbschema, after setting a `replicaCount: 0` in the relevant yaml file, this will then rebuild db with no data, after this set the count to 1 again and merge this new pr and the pod will repop with new data.


### Useful things to find
- EFS ID - found under https://apak.awsapps.com/start under the EFS
- Instance ID - found under EC2
- User for temp role -> found in the `master infrastructure -> users_<location>.tf`
- When making a PR, if you add commits on top it won't automatically build again you need to comment `atlantis plan` to call the webhook to get the build running again
- `name` variable in `modules` in prod, seemed to have an issue with `"sfpw-resiliency-testing-file-failure-efs"` but not `"sfpw-resiliency-testing-etps-2470"`

### Tests
- Node failure, then db failure
- Node failure and db failure together
- Naughty node
- File Storage - PiTR
    - Mainly about creating a `resiliency-test-recovery.yaml` file in the sandbox and updating the `meta/helm.name` to reflect the name, `image.repository` to the correct region, the `fsid` to the new instance id, and the pvcs to be updated through checking the `argo-<region>-applications` under `sandbox resources`
    ```yaml
    meta/helm:
    name: resiliency-test-recovery
    chart: apak/restore-pvc-info
    version: 0.0.1
    helmVersion: 2

    image:
    repository: 493661126902.dkr.ecr.us-east-2.amazonaws.com/apak/restore-pvc-info
    tag: 0.0.4

    fsid: fs-065b43a5bfefe122c
    uid: 9191 #incorrect should've been just 91 (same below)
    gid: 9191

    pvcs:
    - name: sandbox-test-resiliency-pvc
        original: pvc-dea5251b-8c60-4a77-a511-aae5a6b33255
    ```
    This failed, Gerald then had to merge in changes to the `sandbox.yaml` in the `USE2P` repo, which would allow PersistentVolumes to be added to the argo pipeline (were originally blacklisted bc can be used maliciously) for the purpose of this test.

```yaml
meta/helm:
  name: efs-jira-restored-volume
  chart: apak/restore-pvc-info
  version: 0.2.0
  helmVersion: 2

image:
  repository: 493661126902.dkr.ecr.eu-west-1.amazonaws.com/apak/restore-pvc-info
  tag: 0.2.1

fsid: fs-065b43a5bfefe122c
uid: 91
gid: 91
datasyncFsid: fs-04bdad4bfee5d79d7
owner: "Max Fudge"
ownerFunction: "Product Management"
region: "us-east-2"
stage: "test"
account: "493661126902"
scheduleMinutes: 30

pvcs:
  - name: sandbox-test-resiliency-etps-2470-pvc
    original: pvc-dea5251b-8c60-4a77-a511-aae5a6b33255 # volumeName from sandbox-test-resiliency-pvc (in resiliency-test-recover its mapped as sandbox-test-resiliency-recovered-pv)
    target: pvc-b97a28c5-f4c3-45d2-b37c-fdf05a376efc # volumeName from target sandbox-test-resiliency-etps-2470-pvc
```

### Teardown
#### Temporary Role Removal
- Definition of role done in `prod-compliance-tf -> policy.tf` via Terraform
    - Remove the role:
    ```json
    resource "aws_iam_policy" "resiliency_testing_overrides" {
    name        = "resiliency-testing-overrides"
    path        = "/"
    description = "Policy to allow SFPW resiliency testing."
    policy      = data.aws_iam_policy_document. resiliency_testing.json
    }

    data "aws_iam_policy_document" "resiliency_testing" {

    statement {
        sid = "AllowDatabaseFailover"
        actions = [
        "rds:FailoverDBCluster",
    ]
        effect = "Allow"
        resources = ["arn:aws:rds:us-east-2:493661126902:cluster:sfpw-resiliency-testing-cluster"]
    }

    statement {
        sid = "TerminateInstance"
        actions = [
        "ec2:TerminateInstances"
        ]
        effect = "Allow"

        condition {
        test     = "StringLike"
        variable = "ec2:InstanceID"
        values   = ["i-014d185bb324397a3"]
        }
        resources = ["*"]
    }
    }
    ```
    And remove the role assignment from `master-infrastructure-tf -> cloud_native_role_assignments.tf`
    ```json
    module "resiliency_testing" {
    source = "./role"

    identity_store_arn = local.master_identity_store_arn
    name = "Resiliency-Testing"
    }

    resource "aws_ssoadmin_account_assignment" "alex_jones3_resiliency_testing" {
    instance_arn = local.master_identity_store_arn
    permission_set_arn = module.resiliency_testing.role_permission_set_arn

    principal_id = module.alex_jones3_soprabanking_com.id
    principal_type = "USER"

    target_id = aws_organizations_account.apak_production.id
    target_type = "AWS_ACCOUNT"
    }
    ```
    Cloud role assignments should be merged first as it actually assigns the temporary admin role, then the policy should be merged. To monitor this, go into `AWS -> CodeBuild -> EU-West-1` (eu despite doing in OHIO as this is where we centralise), then wait for job to finish
-	Remove the SFP-W instance. This needs to be done in this order (due to dependencies between the resources)
    - Remove applications/resiliency-test.yaml and resources/resiliency-test-schema.yaml Merge and sync (with prune) both in Argo
    - Remove resources/ resiliency-test-database-secret.yaml and resources/sfpw-resiliency-testing-admin.yaml Merge and sync (with prune) both in Argo
    - All other resiliency test files Merge and sync (with prune) resources in Argo
-	Shutdown the:
    - Database (This is a two step process): 
        - Set skip_final_snapshot and deletion_protection to “false” Get this change approved and merge, 
        - Do a PR to remove the database resiliency-test
    - The resiliency-test worker Node
    - resiliency-test 
    - The EFS + DataSync that got created as part of recovery – See the recovery document on how to do this
-	Remove the Storage Class from cluster-config
- Remove hidden storage backup from prod-us-east-2 : Part of [this README](https://bitbucket.apak.delivery/projects/IN/repos/documentation/browse/filesystem-recovery/restoring_an_efs_backup.md)
```yaml
 build:
    on-failure: ABORT
    commands:
      - echo Entered the build phase...
      - cd ${CODEBUILD_SRC_DIR}
      # Retrieve Bitbucket SSH key for TF Modules 
      # and a Vault Token (which will auto renew if it expires) 
      - nohup python ./scripts/vault-auth.py --
      - python ./scripts/vault-auth.py --wait # wait for the call above to complete
      - terraform init
      - terraform import module.restored-efs.aws_efs_file_system.efs-fs fs-0ae02ed2e771b4075
      - terraform import module.restored-efs.aws_efs_mount_target.efs_mount[0] 	fsmt-08573d2d1c6a35402
      - terraform import module.restored-efs.aws_efs_mount_target.efs_mount[1] 	fsmt-069d6b647ad315123
      - terraform import module.restored-efs.aws_efs_mount_target.efs_mount[2] fsmt-0882911537fbb52bd
      - terraform apply -auto-approve
    finally:
      - python ./scripts/vault-auth.py --stop
```
