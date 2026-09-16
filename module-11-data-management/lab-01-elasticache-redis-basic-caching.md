# Lab 11.1: ElastiCache Redis Basic Caching

**Module:** 11 — Cloud Native Data Management on AWS
**Difficulty:** Intermediate
**Duration:** ~15 minutes
**Cost:** ⚠️ **NOT Free Tier.** ElastiCache has no perpetual free tier (a limited-time free trial exists only for brand-new accounts on `cache.t2.micro`/`cache.t3.micro` for 750 hours in the first 12 months — check your eligibility in the console before starting). Even the smallest node (`cache.t4g.micro`) costs roughly **$0.016/hour (~$0.38/day)** if outside the trial. This lab is designed to be completed and torn down within 15–20 minutes to minimize any charge.

## Objective
Stand up a minimal single-node ElastiCache for Redis cluster and demonstrate the caching pattern: check cache → miss → fetch from "database" → populate cache → next read is a hit.

## Prerequisites
- Completed Lab 1.1
- A default VPC in `us-east-1` (most accounts have one automatically) — check with:
  ```bash
  aws ec2 describe-vpcs --filters "Name=isDefault,Values=true" --region us-east-1
  ```

## Steps

1. **Create a cache subnet group** using your default VPC's subnets:
   ```bash
   SUBNET_IDS=$(aws ec2 describe-subnets --filters "Name=default-for-az,Values=true" --query 'Subnets[*].SubnetId' --output text --region us-east-1)
   aws elasticache create-cache-subnet-group \
     --cache-subnet-group-name demo-cache-subnet-group \
     --cache-subnet-group-description "Demo subnet group" \
     --subnet-ids $SUBNET_IDS
   ```
2. **Create a security group allowing Redis port 6379 from your IP only:**
   ```bash
   VPC_ID=$(aws ec2 describe-vpcs --filters "Name=isDefault,Values=true" --query 'Vpcs[0].VpcId' --output text)
   SG_ID=$(aws ec2 create-security-group --group-name redis-demo-sg --description "Redis demo access" --vpc-id $VPC_ID --query 'GroupId' --output text)
   MY_IP=$(curl -s ifconfig.me)
   aws ec2 authorize-security-group-ingress --group-id $SG_ID --protocol tcp --port 6379 --cidr ${MY_IP}/32
   ```
3. **Create the smallest possible Redis cluster:**
   ```bash
   aws elasticache create-cache-cluster \
     --cache-cluster-id demo-redis-cluster \
     --engine redis \
     --cache-node-type cache.t4g.micro \
     --num-cache-nodes 1 \
     --cache-subnet-group-name demo-cache-subnet-group \
     --security-group-ids $SG_ID \
     --region us-east-1
   ```
   This takes **5–10 minutes** to become available.
4. **Wait for it to be ready:**
   ```bash
   aws elasticache wait cache-cluster-available --cache-cluster-id demo-redis-cluster
   ```
5. **Get the endpoint:**
   ```bash
   ENDPOINT=$(aws elasticache describe-cache-clusters --cache-cluster-id demo-redis-cluster --show-cache-node-info --query 'CacheClusters[0].CacheNodes[0].Endpoint.Address' --output text)
   echo $ENDPOINT
   ```
6. **Connect and test the cache-aside pattern** using `redis-cli` (install via `brew install redis` on Mac, `apt install redis-tools` on Linux, or use `redis-cli` bundled with Docker: `docker run -it --rm redis redis-cli -h <endpoint>`):
   ```bash
   redis-cli -h $ENDPOINT -p 6379
   ```
   Inside the redis-cli prompt:
   ```
   GET user:1001
   SET user:1001 "{\"name\":\"Jane Doe\",\"email\":\"jane@example.com\"}" EX 60
   GET user:1001
   ```

## Expected Result / Validation
- The first `GET user:1001` should return `(nil)` — simulating a cache miss (in a real app, this would trigger a database query).
- After `SET`, the second `GET user:1001` should return the JSON string you stored — simulating a cache hit, avoiding a repeat database call. The `EX 60` sets a 60-second TTL, demonstrating cache expiration.

## Cleanup
**Delete this immediately after validating, given the per-hour cost:**
```bash
aws elasticache delete-cache-cluster --cache-cluster-id demo-redis-cluster
aws elasticache wait cache-cluster-deleted --cache-cluster-id demo-redis-cluster
aws elasticache delete-cache-subnet-group --cache-subnet-group-name demo-cache-subnet-group
aws ec2 delete-security-group --group-id $SG_ID
```
Verify:
```bash
aws elasticache describe-cache-clusters --cache-cluster-id demo-redis-cluster
```
Should return a `CacheClusterNotFound` error.

## Troubleshooting
- **Connection timeout from `redis-cli`** → Confirm your current public IP still matches what was authorized in step 2 (IPs can change); re-run `curl -s ifconfig.me` and update the security group rule if needed.
- **`create-cache-cluster` fails referencing subnet group** → Confirm your account has a default VPC; if not, either create one or adapt the subnet group to a custom VPC's subnet IDs.
- **Security group deletion fails ("in use")** → Wait a minute after the cache cluster fully deletes before deleting the security group; ENIs can take a short time to detach.
