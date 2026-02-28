---
name: analyze
description: Analyzes an existing Node.js/Express app and creates a migration plan. Use when the user wants to migrate, convert, or deploy their app.
---

# Analyze — Understand Your App

You are analyzing the user's existing Node.js/Express application to create a migration plan for AWS serverless deployment.

## When This Skill Activates
- User says "I want to deploy this to AWS"
- User asks to migrate, convert, or move their app to the cloud
- User asks Claude to look at their app for deployment
- No `.migration/plan.md` exists yet

## What To Do

### Phase 1: Explore

Thoroughly read the existing application:

1. **Find the entry point** — Look for `app.js`, `index.js`, `server.js`, or `package.json` main field
2. **Map all routes** — Find every Express route (GET, POST, PUT, DELETE, PATCH)
   - Note the HTTP method, path, and what each handler does
   - Identify middleware (auth checks, body parsing, validation)
3. **Find static files** — Look for `express.static()` calls, HTML/CSS/JS files, assets
4. **Detect data patterns** — How does the app store data?
   - In-memory variables/arrays
   - JSON file reads/writes
   - SQLite or other local database
   - External API calls
5. **Check for auth** — Does the app have login/signup? Sessions? JWT?
6. **Read package.json** — Note dependencies, scripts, Node.js version
7. **Check for environment variables** — `.env` files, `process.env` usage

### Phase 2: Plan

Present findings to the user in plain language:

**Routes Summary:**
"I found [N] Express routes. Here's what each one does:"
- `GET /` — Serves the main page
- `GET /api/items` — Returns a list of items from [data source]
- `POST /api/items` — Creates a new item
- etc.

**Data Storage:**
"Your app currently stores data in [method]. In AWS, we'll move this to DynamoDB, a cloud database that scales automatically."

**Static Files:**
"You have [N] HTML files and [N] CSS/JS files. We'll migrate these into a modern React frontend built with Vite, then host the production build on CloudFront, a global CDN."

**Authentication:**
"Your app [does/doesn't] have authentication. [If yes: We'll move this to Cognito. If no: We can add it later if you need it.]"

### Phase 3: Discuss

Ask the user targeted questions:

1. "Does this summary look right? Did I miss anything?"
2. "Which routes are the most important to get working first?"
3. "Are there any routes that should only be accessible to logged-in users?"
4. "Is there any data that needs to persist between sessions?" (if unclear)
5. "Are there any features you want to add or remove during the migration?"
6. "Do you have an AWS account set up? (If not, I can help with that)"

### Phase 4: Output

Create `.migration/plan.md` with:

```markdown
# Migration Plan

## App Summary
- **Entry point**: [file]
- **Framework**: Express [version]
- **Node.js version**: [version]

## Routes to Migrate
| Method | Path | Description | Auth Required | Lambda Handler |
|--------|------|-------------|---------------|----------------|
| GET | / | Main page | No | N/A (static) |
| GET | /api/items | List items | No | get-items |
| POST | /api/items | Create item | Yes | create-item |

## Data Model
- **Current storage**: [method]
- **Target**: DynamoDB
- **Tables needed**: [list with key design]

## Frontend
- **Current files**: [list of HTML/CSS/JS files]
- **Target**: Vite + React (TypeScript) → built to static files → hosted on S3 + CloudFront

## Authentication
- **Needed**: Yes/No
- **Protected routes**: [list]

## Migration Steps
1. ✅ Analyze — Complete
2. 🔲 Scaffold — Create CDK project
3. 🔲 Create API — [N] Lambda handlers needed
4. 🔲 Add Database — [if needed]
5. 🔲 Add Auth — [if needed]
6. 🔲 Setup Frontend — Create Vite + React app, host on S3 + CloudFront
7. 🔲 Deploy
8. 🔲 Test
```

### Phase 5: Verify

- Confirm the plan with the user
- Explain the next step (scaffold) and what it will create
- Tell them they can just say "let's set up the infrastructure" to continue

## Important Notes

- Use the AWS Documentation MCP to look up serverless patterns when explaining the migration
- If the app has patterns you're unsure about, ask the user rather than guessing
- If the app is very complex, suggest starting with the core routes and adding features incrementally
