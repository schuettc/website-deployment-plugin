---
name: add-database
description: Adds DynamoDB database persistence to the application. Use when the app needs data storage or persistence.
---

# Add Database — DynamoDB Persistence

You are adding DynamoDB to replace the app's current data storage (in-memory, JSON files, etc.).

## Prerequisites
- `.migration/plan.md` exists with data model information
- Lambda handlers exist (from create-api step)

## What To Do

### Phase 1: Explore

Examine how the app currently stores data (in-memory arrays, JSON files, SQLite, etc.). Identify all data entities and their relationships.

### Phase 2: Plan

Present the proposed table design:
```
Table: Items
├── Partition Key: id (String)
├── Sort Key: (if needed based on access patterns)
├── Attributes: name, description, createdAt, updatedAt
└── Billing: Pay-per-request
```

Ask the user:
1. "Does this table design capture all your data?"
2. "How will you typically look up data? By ID? By user? By date?" (affects key design)
3. "Do you need to search or filter beyond looking up by ID?" (may need GSI)
4. "Is any data temporary and can be auto-deleted?" (TTL feature)

### Phase 3: Execute

1. **Create `infrastructure/lib/database-stack.ts`**
   - DynamoDB table with:
     - Partition key and optional sort key
     - Pay-per-request billing mode
     - Point-in-time recovery enabled
     - Encryption at rest (default)
     - RemovalPolicy.RETAIN
   - Global Secondary Indexes if needed
   - Export table name and ARN for other stacks

2. **Update Lambda handlers** to use DynamoDB:
   - Add `@aws-sdk/client-dynamodb` and `@aws-sdk/lib-dynamodb` to Lambda dependencies
   - Create shared DynamoDB client in `infrastructure/lambda/shared/db.ts`
   - Update each handler to use `PutCommand`, `GetCommand`, `QueryCommand`, etc.
   - Add table name as environment variable to each Lambda
   - Show before/after for at least one handler

3. **Update CDK stack** with least-privilege permissions:
   - Read handlers: `dynamodb:GetItem`, `dynamodb:Query`
   - Write handlers: `dynamodb:PutItem`, `dynamodb:UpdateItem`
   - Delete handlers: `dynamodb:DeleteItem`
   - Pass table name as environment variable

### Phase 4: Verify

1. Run `cd infrastructure && npx cdk synth` to verify compilation
2. Show the user: table definition, updated handler, IAM permissions granted
3. Update `.migration/plan.md` to mark add-database as complete

## Important Notes

### Best Practices
- Always use pay-per-request billing for new projects
- Design keys around access patterns, not just data structure
- Use `DynamoDBDocumentClient` for simpler syntax
- Always include error handling for DynamoDB operations
- Table names passed via environment variables, never hardcoded

### Security
- Grant least-privilege: read-only handlers get read-only access
- Never grant `dynamodb:*` — list specific actions

### Cost
- Pay-per-request: first 25 GB free, reads/writes are pennies
- Effectively free for development and low-traffic apps
