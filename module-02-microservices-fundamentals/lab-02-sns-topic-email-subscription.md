# Lab 2.2: Amazon SNS — Topic + Email Subscription

**Module:** 2 — Microservices Architecture Fundamentals
**Difficulty:** Beginner
**Duration:** ~15 minutes
**Cost:** ✅ Free Tier Eligible (1,000 email notifications/month free — this lab uses 1–2)

## Objective
Understand the publish/subscribe (fan-out) pattern used to broadcast events to multiple microservices, using SNS with an email subscriber standing in for a downstream service.

## Prerequisites
- Completed Lab 2.1
- Access to a real email inbox you can check during the lab

## Steps

1. **Create an SNS topic**:
   ```bash
   aws sns create-topic --name order-notifications --region us-east-1
   ```
   Copy the `TopicArn` from the output.

2. **Store it in a variable**:
   ```bash
   TOPIC_ARN=$(aws sns list-topics --query "Topics[?contains(TopicArn, 'order-notifications')].TopicArn" --output text)
   echo $TOPIC_ARN
   ```

3. **Subscribe your email address**:
   ```bash
   aws sns subscribe \
     --topic-arn $TOPIC_ARN \
     --protocol email \
     --notification-endpoint your-email@example.com
   ```
   Replace `your-email@example.com` with a real inbox you can access.

4. **Confirm the subscription**
   - Check your email inbox for a message titled "AWS Notification - Subscription Confirmation."
   - Click **Confirm subscription** in that email.

5. **Publish a message** to the topic (simulating a microservice broadcasting an event):
   ```bash
   aws sns publish \
     --topic-arn $TOPIC_ARN \
     --subject "Order Shipped" \
     --message "Order #1001 has shipped and is on its way."
   ```

## Expected Result / Validation
Within a minute or two, you should receive an email with the subject "Order Shipped" and the message body from step 5. This demonstrates fan-out: in a real system, this same topic could simultaneously notify an email service, a Lambda function, and an SQS queue — all decoupled from the publisher.

## Cleanup
1. Unsubscribe (optional, deleting the topic removes subscriptions too):
   ```bash
   aws sns list-subscriptions-by-topic --topic-arn $TOPIC_ARN
   ```
2. Delete the topic:
   ```bash
   aws sns delete-topic --topic-arn $TOPIC_ARN
   ```
3. Verify:
   ```bash
   aws sns list-topics
   ```
   `order-notifications` should no longer appear.

## Troubleshooting
- **No confirmation email arrives** → Check spam/junk folder; confirmation emails can take a few minutes.
- **`publish` succeeds but no email received** → Make sure you clicked "Confirm subscription" — unconfirmed subscriptions silently drop messages.
