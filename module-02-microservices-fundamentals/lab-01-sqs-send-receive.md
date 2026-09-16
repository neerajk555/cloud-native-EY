# Lab 2.1: Amazon SQS — Send & Receive Messages

**Module:** 2 — Microservices Architecture Fundamentals
**Difficulty:** Beginner
**Duration:** ~15 minutes
**Cost:** ✅ Free Tier Eligible (1 million SQS requests/month free — this lab uses a handful)

## Objective
Understand asynchronous communication between microservices by creating a Standard SQS queue and manually sending/receiving messages, simulating how one service hands off work to another.

## Prerequisites
- Completed Module 1, Lab 1.1 (CLI configured, region `us-east-1`)

## Steps

1. **Create a queue** via CLI:
   ```bash
   aws sqs create-queue --queue-name order-events-queue --region us-east-1
   ```
   Note the `QueueUrl` returned in the output — you'll need it below.

2. **Store the queue URL** in a variable for convenience:
   ```bash
   QUEUE_URL=$(aws sqs get-queue-url --queue-name order-events-queue --query 'QueueUrl' --output text)
   echo $QUEUE_URL
   ```

3. **Send a message** (simulating an "Order Service" publishing an event):
   ```bash
   aws sqs send-message \
     --queue-url $QUEUE_URL \
     --message-body '{"orderId": "1001", "status": "CREATED"}'
   ```

4. **Send a second message**:
   ```bash
   aws sqs send-message \
     --queue-url $QUEUE_URL \
     --message-body '{"orderId": "1002", "status": "CREATED"}'
   ```

5. **Receive messages** (simulating a "Fulfillment Service" consuming events):
   ```bash
   aws sqs receive-message --queue-url $QUEUE_URL --max-number-of-messages 5
   ```
   Note the `ReceiptHandle` value in the response — it's required to delete the message.

6. **Delete a processed message** (replace `<ReceiptHandle>` with the value from step 5):
   ```bash
   aws sqs delete-message --queue-url $QUEUE_URL --receipt-handle "<ReceiptHandle>"
   ```

## Expected Result / Validation
- Step 5 should return the JSON bodies of both messages you sent.
- After step 6, re-run the receive command — the deleted message should no longer appear (the other one will still be there until you delete it too).

## Cleanup
Delete the queue entirely so no messages or infrastructure linger:
```bash
aws sqs delete-queue --queue-url $QUEUE_URL
```
Verify it's gone:
```bash
aws sqs list-queues
```
`order-events-queue` should not appear in the output.

## Troubleshooting
- **`QueueDoesNotExist`** → Double check the region is `us-east-1` and the queue name is spelled exactly `order-events-queue`.
- **Receive returns nothing** → SQS uses short polling by default; simply re-run the receive-message command, or add `--wait-time-seconds 5` for long polling.
