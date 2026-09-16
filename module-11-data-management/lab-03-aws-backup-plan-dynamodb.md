# Lab 11.3: AWS Backup Plan for DynamoDB

**Module:** 11 — Cloud Native Data Management on AWS
**Difficulty:** Beginner
**Duration:** ~15 minutes
**Cost:** ✅ Free Tier Eligible (AWS Backup pricing is pay-per-GB backed up; a tiny demo table with a few KB of data costs a negligible fraction of a cent — effectively free for this lab's scope)

## Objective
Create a DynamoDB table, configure an AWS Backup plan and vault to protect it on a schedule, and trigger an on-demand backup to see the mechanism work immediately (rather than waiting for the schedule).

## Prerequisites
- Completed Lab 1.1

## Steps

1. **Create a small DynamoDB table** (or reuse the `Orders` table concept from Module 2, Lab 2.4 — this creates a fresh one):
   ```bash
   aws dynamodb create-table \
     --table-name BackupDemoTable \
     --attribute-definitions AttributeName=Id,AttributeType=S \
     --key-schema AttributeName=Id,KeyType=HASH \
     --billing-mode PAY_PER_REQUEST \
     --region us-east-1
   aws dynamodb wait table-exists --table-name BackupDemoTable
   aws dynamodb put-item --table-name BackupDemoTable --item '{"Id": {"S": "1"}, "Note": {"S": "Important data to protect"}}'
   ```
2. **Create a backup vault** (a logical container for backups):
   ```bash
   aws backup create-backup-vault --backup-vault-name demo-backup-vault --region us-east-1
   ```
3. **Create an IAM role for AWS Backup:**
   ```bash
   aws iam create-role \
     --role-name AWSBackupDemoRole \
     --assume-role-policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"Service":"backup.amazonaws.com"},"Action":"sts:AssumeRole"}]}'
   aws iam attach-role-policy --role-name AWSBackupDemoRole --policy-arn arn:aws:iam::aws:policy/service-role/AWSBackupServiceRolePolicyForBackup
   BACKUP_ROLE_ARN=$(aws iam get-role --role-name AWSBackupDemoRole --query 'Role.Arn' --output text)
   ```
4. **Create a backup plan** — `backup-plan.json` — scheduled daily but we'll trigger an on-demand backup for immediate validation:
   ```json
   {
     "BackupPlanName": "demo-daily-backup-plan",
     "Rules": [
       {
         "RuleName": "DailyBackups",
         "TargetBackupVaultName": "demo-backup-vault",
         "ScheduleExpression": "cron(0 5 * * ? *)",
         "StartWindowMinutes": 60,
         "CompletionWindowMinutes": 120,
         "Lifecycle": { "DeleteAfterDays": 7 }
       }
     ]
   }
   ```
   ```bash
   PLAN_ID=$(aws backup create-backup-plan --backup-plan file://backup-plan.json --query 'BackupPlanId' --output text)
   ```
5. **Assign the DynamoDB table as a backup resource:**
   ```bash
   TABLE_ARN=$(aws dynamodb describe-table --table-name BackupDemoTable --query 'Table.TableArn' --output text)
   ```
   ```json
   {
     "BackupPlanId": "PLAN_ID_PLACEHOLDER",
     "BackupSelection": {
       "SelectionName": "demo-table-selection",
       "IamRoleArn": "ROLE_ARN_PLACEHOLDER",
       "Resources": ["TABLE_ARN_PLACEHOLDER"]
     }
   }
   ```
   Save as `selection.json`, then substitute values and apply:
   ```bash
   sed -i "s|PLAN_ID_PLACEHOLDER|$PLAN_ID|g; s|ROLE_ARN_PLACEHOLDER|$BACKUP_ROLE_ARN|g; s|TABLE_ARN_PLACEHOLDER|$TABLE_ARN|g" selection.json
   aws backup create-backup-selection --backup-plan-id $PLAN_ID --backup-selection file://selection.json
   ```
6. **Trigger an on-demand backup immediately** (don't wait for the 5am cron schedule):
   ```bash
   aws backup start-backup-job \
     --backup-vault-name demo-backup-vault \
     --resource-arn $TABLE_ARN \
     --iam-role-arn $BACKUP_ROLE_ARN
   ```

## Expected Result / Validation
```bash
aws backup list-backup-jobs --by-backup-vault-name demo-backup-vault
```
You should see a job with `State: CREATED` or `RUNNING`, progressing to `COMPLETED` within a few minutes. Once complete:
```bash
aws backup list-recovery-points-by-backup-vault --backup-vault-name demo-backup-vault
```
This should show one recovery point for `BackupDemoTable`, confirming a point-in-time backup now exists and could be restored if needed.

## Cleanup
```bash
RECOVERY_POINT_ARN=$(aws backup list-recovery-points-by-backup-vault --backup-vault-name demo-backup-vault --query 'RecoveryPoints[0].RecoveryPointArn' --output text)
aws backup delete-recovery-point --backup-vault-name demo-backup-vault --recovery-point-arn $RECOVERY_POINT_ARN
aws backup delete-backup-selection --backup-plan-id $PLAN_ID --selection-id $(aws backup list-backup-selections --backup-plan-id $PLAN_ID --query 'BackupSelectionsList[0].SelectionId' --output text)
aws backup delete-backup-plan --backup-plan-id $PLAN_ID
aws backup delete-backup-vault --backup-vault-name demo-backup-vault
aws iam detach-role-policy --role-name AWSBackupDemoRole --policy-arn arn:aws:iam::aws:policy/service-role/AWSBackupServiceRolePolicyForBackup
aws iam delete-role --role-name AWSBackupDemoRole
aws dynamodb delete-table --table-name BackupDemoTable
```
Verify:
```bash
aws backup list-backup-plans --query "BackupPlansList[?BackupPlanName=='demo-daily-backup-plan']"
```
Should return an empty list.

## Troubleshooting
- **Recovery point deletion fails ("recovery point is not deletable yet")** → Backups sometimes remain in a transitional state briefly after job completion; wait 1–2 minutes and retry.
- **`create-backup-selection` fails with role errors** → Wait 10–15 seconds after IAM role creation before referencing it, due to IAM eventual consistency.
