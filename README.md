# AWS Website Deployment Plugin for Claude Code

A guided workflow plugin that helps you deploy Node.js/Express applications to AWS serverless architecture. Designed for developers new to AWS — Claude explains every concept, asks before acting, and guides you step by step.

## What It Does

This plugin transforms your locally-running Express app into a production-ready AWS serverless application:

```
Your Express App → Lambda + API Gateway + S3 + CloudFront + DynamoDB + Cognito
```

**Before:** Your app runs on `localhost:3000` with Express serving everything.

**After:** Your app runs on AWS with:
- A React frontend (built with Vite) served globally via CloudFront CDN
- API routes running as Lambda functions
- Data stored in DynamoDB (if needed)
- User authentication via Cognito (if needed)
- Everything defined as code with CDK (version-controlled, repeatable)

## Prerequisites

Before installing the plugin, make sure you have:

1. **Node.js 18+** — [Download](https://nodejs.org/)
   ```bash
   node --version  # Should be 18.x or higher
   ```

2. **AWS CLI** — [Installation guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
   ```bash
   aws --version  # Should show aws-cli/2.x.x
   ```

3. **AWS Account & Credentials** — [See AWS Account Setup below](#aws-account-setup)
   ```bash
   aws sts get-caller-identity  # Should show your account ID
   ```

4. **Claude Code 1.0.33+** — [Installation guide](https://code.claude.com/docs/en/quickstart)
   ```bash
   claude --version
   ```

5. **Python & uvx** (for AWS Documentation MCP server)
   ```bash
   uvx --version
   ```

## Installation

### From the marketplace (recommended)

Open Claude Code and run:

```
/plugin marketplace add schuettc/website-deployment-plugin
```

Then install the plugin:

```
/plugin install website-deployment@website-deployment-marketplace
```

### Local development / testing

Clone the repo and load it directly:

```bash
git clone https://github.com/schuettc/website-deployment-plugin.git
claude --plugin-dir ./website-deployment-plugin/plugin
```

## Quick Start

1. **Open your Express project** in Claude Code (with the plugin loaded):
   ```bash
   cd your-express-app
   claude
   ```

2. **Tell Claude you want to deploy:**
   ```
   I want to deploy this app to AWS
   ```
   Claude will analyze your app, explain the migration plan, and guide you through each step.

3. **Follow the guided workflow.** Claude will:
   - Analyze your app and show you what it found
   - Explain each AWS service in simple terms
   - Ask your preferences before making changes
   - Create infrastructure code step by step
   - Build a React frontend with Vite from your existing HTML/CSS/JS
   - Deploy when you're ready

4. **Check progress anytime:**
   ```
   /website-deployment:guide
   ```
   Shows where you are in the migration and what to do next.

## Workflow

The migration follows these steps. Most activate automatically — Claude detects what you need:

| Step | Skill | How It Activates |
|------|-------|-----------------|
| 1 | **Guide** | Auto — when you ask "what's next?" or seem lost |
| 2 | **Analyze** | Auto — when you say "deploy to AWS" or "migrate my app" |
| 3 | **Scaffold** | Auto — when setting up infrastructure |
| 4 | **Create API** | Auto — when converting routes to Lambda |
| 5 | **Add Database** | Auto — when your app needs data persistence |
| 6 | **Add Auth** | Auto — when your app needs authentication |
| 7 | **Setup Frontend** | Auto — when setting up React + static file hosting |
| 8 | **Deploy** | Manual — run `/website-deployment:deploy` explicitly |
| 9 | **Test** | Auto — when you want to verify the deployment |
| 10 | **Teardown** | Manual — run `/website-deployment:teardown` explicitly |

Steps 5 and 6 are optional — they're only suggested if your app needs them.

**Only two commands require explicit invocation:**
- `/website-deployment:deploy` — Creates real AWS resources (may incur costs)
- `/website-deployment:teardown` — Permanently deletes AWS resources

Everything else flows naturally through conversation.

## Skill Reference

| Skill | Description |
|-------|-------------|
| `/website-deployment:guide` | Show migration progress and next steps |
| `/website-deployment:analyze` | Analyze your app and create a migration plan |
| `/website-deployment:scaffold` | Create the CDK infrastructure project |
| `/website-deployment:create-api` | Convert Express routes to Lambda + API Gateway |
| `/website-deployment:add-database` | Add DynamoDB persistence |
| `/website-deployment:add-auth` | Add Cognito authentication |
| `/website-deployment:setup-frontend` | Create Vite + React app, host on S3 + CloudFront |
| `/website-deployment:deploy` | Deploy to AWS (creates real resources) |
| `/website-deployment:test` | Run end-to-end tests with Playwright |
| `/website-deployment:teardown` | Remove all AWS resources |

## AWS Account Setup

If you don't have an AWS account yet:

1. **Create an account** at [aws.amazon.com](https://aws.amazon.com/)
   - You'll need a credit card, but most services have a generous free tier

2. **Create an IAM user** for CLI access:
   - Go to IAM Console -> Users -> Create User
   - Attach the `AdministratorAccess` policy (for development)
   - Create an access key for CLI usage

3. **Configure the CLI:**
   ```bash
   aws configure
   ```
   Enter your access key, secret key, and preferred region (e.g., `us-east-1`).

4. **Verify it works:**
   ```bash
   aws sts get-caller-identity
   ```

> **Security note:** For production use, create a user with more restricted permissions. `AdministratorAccess` is fine for learning and development.

## Cost Expectations

Most resources this plugin creates are **free-tier eligible**:

| Service | Free Tier (12 months) |
|---------|----------------------|
| Lambda | 1M requests/month, 400,000 GB-seconds |
| API Gateway | 1M API calls/month |
| DynamoDB | 25 GB storage, 25 read/write capacity units |
| S3 | 5 GB storage, 20,000 GET requests |
| CloudFront | 1 TB data transfer, 10M requests/month |
| Cognito | 50,000 monthly active users |

**For a development/demo app with low traffic, your monthly cost will likely be $0.**

To avoid unexpected charges:
- Run `/website-deployment:teardown` when you're done experimenting
- Set up a [billing alarm](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/monitor_estimated_charges_with_cloudwatch.html) in the AWS Console
- Check the [AWS Free Tier dashboard](https://console.aws.amazon.com/billing/home#/freetier) regularly

## Troubleshooting

### "AWS CLI not configured"
Run `aws configure` and enter your credentials. See [AWS Account Setup](#aws-account-setup).

### "CDK bootstrap required"
Run `/website-deployment:deploy` — it will automatically bootstrap CDK in your account/region.

### "Permission denied" during deploy
Your IAM user may not have sufficient permissions. For development, use `AdministratorAccess`.

### "Resource already exists"
A previous deployment may have left orphaned resources. Check the AWS CloudFormation Console and delete any stuck stacks.

### CloudFront takes a long time
CloudFront distributions take 5-15 minutes to create or update. This is normal.

### Lambda cold starts
The first request after idle may take 1-3 seconds. Subsequent requests are fast. This is normal for Lambda.

### CORS errors in the browser
Tell Claude to check and fix CORS configuration — it will load the create-api skill automatically.

## Cleanup

To remove all AWS resources and stop any charges:

```
/website-deployment:teardown
```

This will:
1. Show you every resource that will be deleted
2. Ask for confirmation
3. Delete all stacks and resources
4. Report any resources that need manual cleanup

**Your local code is never deleted** — you can always redeploy later.

## Architecture

```
                          +-------------+
                          |  CloudFront  |
                          |    (CDN)     |
                          +------+------+
                                 |
                    +------------+------------+
                    |                         |
              /api/* routes          Static files (React)
                    |                         |
           +-------+-------+          +------+------+
           |  API Gateway  |          |  S3 Bucket  |
           |  (REST API)   |          |  (Private)  |
           +-------+-------+          +-------------+
                   |
          +--------+--------+
          |        |        |
     +----+---+ +--+---+ +--+----+
     | Lambda | |Lambda| |Lambda |
     | GET /  | |POST /| |DELETE/|
     +----+---+ +--+---+ +--+----+
          |        |        |
          +--------+--------+
                   |
           +-------+-------+
           |   DynamoDB    |
           |   (Database)  |
           +---------------+
```

## Contributing

Contributions welcome! Please open an issue or PR on [GitHub](https://github.com/schuettc/website-deployment-plugin).

## License

MIT
