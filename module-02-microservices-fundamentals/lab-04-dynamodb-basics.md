# Lab 2.4: DynamoDB Basics — Database-per-Service

**Module:** 2 — Microservices Architecture Fundamentals
**Difficulty:** Beginner
**Duration:** ~15 minutes
**Cost:** ✅ Free Tier Eligible (25 GB storage + 25 RCU/WCU always free)

## Objective
Create a DynamoDB table representing a single microservice's private data store, and perform basic CRUD operations, illustrating the "database per service" pattern.

## Prerequisites
- Completed Lab 1.1

## Steps

1. **Create a table** for an "Orders" microservice:
   ```bash
   aws dynamodb create-table \
     --table-name Orders \
     --attribute-definitions AttributeName=OrderId,AttributeType=S \
     --key-schema AttributeName=OrderId,KeyType=HASH \
     --billing-mode PAY_PER_REQUEST \
     --region us-east-1
   ```
2. **Wait for it to become active:**
   ```bash
   aws dynamodb wait table-exists --table-name Orders
   ```
3. **Insert an item (Create):**
   ```bash
   aws dynamodb put-item \
     --table-name Orders \
     --item '{"OrderId": {"S": "1001"}, "Status": {"S": "CREATED"}, "Amount": {"N": "49.99"}}'
   ```
4. **Insert a second item:**
   ```bash
   aws dynamodb put-item \
     --table-name Orders \
     --item '{"OrderId": {"S": "1002"}, "Status": {"S": "SHIPPED"}, "Amount": {"N": "129.50"}}'
   ```
5. **Read an item (Read):**
   ```bash
   aws dynamodb get-item --table-name Orders --key '{"OrderId": {"S": "1001"}}'
   ```
6. **Update an item's status (Update):**
   ```bash
   aws dynamodb update-item \
     --table-name Orders \
     --key '{"OrderId": {"S": "1001"}}' \
     --update-expression "SET #s = :newStatus" \
     --expression-attribute-names '{"#s": "Status"}' \
     --expression-attribute-values '{":newStatus": {"S": "SHIPPED"}}'
   ```
7. **Scan the whole table** (see all orders — like a service querying its own private data):
   ```bash
   aws dynamodb scan --table-name Orders
   ```

## Expected Result / Validation
The scan in step 7 should return both items, with `OrderId 1001` now showing `Status: SHIPPED` (confirming the update worked).

## Cleanup
```bash
aws dynamodb delete-table --table-name Orders
```
Verify:
```bash
aws dynamodb list-tables
```
`Orders` should not appear in the list.

## Troubleshooting
- **`ResourceNotFoundException` on get/put** → Make sure `wait table-exists` completed before running further commands.
- **Update expression error** → DynamoDB reserves the word `Status` in some contexts — that's why this lab uses the `#s` expression attribute name alias; keep that syntax if copy-pasting.
