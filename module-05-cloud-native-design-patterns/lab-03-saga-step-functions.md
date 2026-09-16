# Lab 5.3: Saga Pattern with AWS Step Functions

**Module:** 5 — Cloud Native Design Patterns
**Difficulty:** Intermediate
**Duration:** ~15 minutes
**Cost:** ✅ Free Tier Eligible (Step Functions: 4,000 free state transitions/month, always free)

## Objective
Implement a simplified **Saga pattern** using AWS Step Functions: a multi-step "order" workflow that, on failure, runs compensating (rollback) actions instead of a traditional all-or-nothing database transaction.

## Prerequisites
- Completed Lab 1.1 (CLI configured)

## Steps

1. **Create a Step Functions state machine definition** — `saga.asl.json` — that simulates: Reserve Inventory → Charge Payment → (intentionally fails) → Compensate by Releasing Inventory:
   ```json
   {
     "Comment": "Simplified Saga pattern demo",
     "StartAt": "ReserveInventory",
     "States": {
       "ReserveInventory": {
         "Type": "Pass",
         "Result": "Inventory reserved for order #1001",
         "ResultPath": "$.inventoryStatus",
         "Next": "ChargePayment"
       },
       "ChargePayment": {
         "Type": "Pass",
         "Result": "Payment charge FAILED - insufficient funds",
         "ResultPath": "$.paymentStatus",
         "Next": "PaymentSucceeded?"
       },
       "PaymentSucceeded?": {
         "Type": "Choice",
         "Choices": [
           {
             "Variable": "$.simulateFailure",
             "BooleanEquals": true,
             "Next": "CompensateReleaseInventory"
           }
         ],
         "Default": "OrderComplete"
       },
       "CompensateReleaseInventory": {
         "Type": "Pass",
         "Result": "Compensating transaction: inventory released back to stock",
         "ResultPath": "$.compensationStatus",
         "Next": "OrderFailed"
       },
       "OrderFailed": {
         "Type": "Fail",
         "Error": "PaymentFailed",
         "Cause": "Saga rolled back via compensating transaction"
       },
       "OrderComplete": {
         "Type": "Succeed"
       }
     }
   }
   ```
2. **Create an execution role for Step Functions:**
   ```bash
   aws iam create-role \
     --role-name saga-demo-role \
     --assume-role-policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"Service":"states.amazonaws.com"},"Action":"sts:AssumeRole"}]}'
   ROLE_ARN=$(aws iam get-role --role-name saga-demo-role --query 'Role.Arn' --output text)
   ```
3. **Create the state machine:**
   ```bash
   aws stepfunctions create-state-machine \
     --name order-saga-demo \
     --definition file://saga.asl.json \
     --role-arn $ROLE_ARN \
     --region us-east-1
   ```
   Copy the returned `stateMachineArn`.
4. **Start an execution that simulates a payment failure** (triggering the compensating rollback):
   ```bash
   SM_ARN=$(aws stepfunctions list-state-machines --query "stateMachines[?name=='order-saga-demo'].stateMachineArn" --output text)
   aws stepfunctions start-execution \
     --state-machine-arn $SM_ARN \
     --input '{"simulateFailure": true}'
   ```
5. **Check the execution result:**
   ```bash
   aws stepfunctions list-executions --state-machine-arn $SM_ARN
   ```
   Copy the `executionArn`, then:
   ```bash
   aws stepfunctions describe-execution --execution-arn <executionArn>
   ```

## Expected Result / Validation
The execution `status` should show `FAILED` with `error: "PaymentFailed"` — but critically, the execution history (viewable via `aws stepfunctions get-execution-history --execution-arn <executionArn>`) should show that `CompensateReleaseInventory` **did run** before the workflow failed. This is the Saga pattern in action: instead of an atomic rollback like a database transaction, each prior step's effect is explicitly undone by a compensating step.

## Cleanup
```bash
aws stepfunctions delete-state-machine --state-machine-arn $SM_ARN
aws iam delete-role --role-name saga-demo-role
```
Verify:
```bash
aws stepfunctions list-state-machines
```
`order-saga-demo` should no longer appear.

## Troubleshooting
- **`AccessDenied` creating the role** → Confirm you're using the `cloudnative-student` IAM user with AdministratorAccess.
- **Role not found when creating the state machine** → Wait 10–15 seconds after role creation before running `create-state-machine`, due to IAM eventual consistency.
- **Want to see the "happy path"?** → Re-run step 4 with `'{"simulateFailure": false}'` — the execution should complete with `status: SUCCEEDED` via the `OrderComplete` state, skipping compensation entirely.
