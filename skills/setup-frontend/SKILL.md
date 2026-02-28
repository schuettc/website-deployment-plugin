---
name: setup-frontend
description: Sets up a Vite + React frontend with S3 and CloudFront hosting. Use when setting up frontend hosting, CDN, static file serving, or creating the React app.
---

# Setup Frontend — Vite + React on S3 + CloudFront

You are creating a Vite + React frontend and setting up S3 + CloudFront hosting. This replaces Express's `express.static()` with a build pipeline and CDN hosting.

## Prerequisites
- API Gateway is configured (from create-api)
- Node.js 18+ installed

## What To Do

### Phase 1: Plan

Present the plan to the user:
1. Create a Vite + React (TypeScript) app that builds to static files in `dist/`
2. Both frontend and API served through one CloudFront distribution (`/api/*` to API Gateway, everything else to S3) — no CORS issues
3. Migrate existing HTML/CSS/JS into React components
4. Ask if they want a custom domain (Route53) or the default CloudFront URL

### Phase 2: Execute

#### Step 1: Create the Vite + React project

```bash
npm create vite@latest frontend -- --template react-ts
cd frontend && npm install
```

Project structure:
```
frontend/
├── index.html
├── public/
│   └── favicon.ico
├── src/
│   ├── main.tsx
│   ├── App.tsx
│   ├── components/
│   ├── pages/
│   ├── hooks/
│   │   └── useApi.ts
│   ├── lib/
│   │   └── api.ts
│   ├── App.css
│   └── index.css
├── vite.config.ts
├── tsconfig.json
└── package.json
```

#### Step 2: Configure Vite

Update `frontend/vite.config.ts`:
```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  server: {
    proxy: {
      '/api': 'http://localhost:3000', // Proxy API calls to Express during development
    },
  },
  build: {
    outDir: 'dist',
    sourcemap: true,
  },
});
```

#### Step 3: Set up API client

Create `frontend/src/lib/api.ts`:
```typescript
const API_BASE = import.meta.env.VITE_API_URL || '';

export async function apiFetch(path: string, options?: RequestInit) {
  const response = await fetch(`${API_BASE}${path}`, {
    ...options,
    headers: {
      'Content-Type': 'application/json',
      ...options?.headers,
    },
  });
  if (!response.ok) {
    throw new Error(`API error: ${response.status}`);
  }
  return response.json();
}
```

Note: `VITE_API_URL` is empty in production (same-origin). Vite injects `VITE_`-prefixed env vars at build time.

#### Step 4: Migrate existing HTML/CSS into React components

- Convert each HTML page into a React component in `src/pages/`
- Move CSS into component-level CSS or `src/index.css`
- Move static assets (images, fonts) into `public/`
- Replace direct DOM manipulation with React state and effects
- Replace `fetch()` calls with the `apiFetch` helper
- Add React Router if the app has multiple pages:
  ```bash
  cd frontend && npm install react-router-dom
  ```

#### Step 5: Create the CDK frontend stack

Create `infrastructure/lib/frontend-stack.ts`:

- **S3 Bucket:**
  - Block ALL public access
  - Enable server-side encryption (AES-256)
  - RemovalPolicy.DESTROY for development
  - Auto-delete objects on stack deletion

- **Origin Access Control (OAC):**
  - Create OAC (newer, recommended over OAI)
  - Grant S3 read-only access to CloudFront only

- **CloudFront Distribution:**
  - Default behavior: S3 origin (Vite build output)
  - Additional behavior: `/api/*` -> API Gateway origin
  - HTTPS only (redirect HTTP -> HTTPS)
  - Default root object: `index.html`
  - SPA routing: Custom error responses — 403 and 404 -> `/index.html` with 200 status (so React Router handles client-side routes)
  - Price class: PriceClass.PRICE_CLASS_100 (US/Europe — cheapest)
  - Enable compression (gzip/brotli)
  - Response headers policy with security headers

- **S3 Deployment:**
  - Use `BucketDeployment` to upload `frontend/dist/`
  - Cache control:
    - `assets/*` (hashed filenames): `max-age=31536000, immutable`
    - `index.html`: `max-age=0, must-revalidate`
  - Invalidate CloudFront cache after deployment

- **Outputs:** CloudFront URL, distribution ID, S3 bucket name

#### Step 6: Add build step

Update the CDK stack or add a build step that:
1. Runs `cd frontend && npm run build` before deploying
2. Sets `VITE_API_URL` if needed (empty string for same-origin)
3. Uploads `frontend/dist/` to S3

#### Step 7: Verify local development

```bash
cd frontend && npm run dev
```
- Verify the app renders at http://localhost:5173
- Verify API proxy works (if Express is running on port 3000)

### Phase 3: Verify

1. Run `cd frontend && npm run build` to verify the Vite build succeeds
2. Run `cd infrastructure && npx cdk synth` to verify CDK compilation
3. Update `.migration/plan.md` to mark setup-frontend as complete

## Important Notes

### Security
- NEVER make the S3 bucket public — always use OAC
- Enforce HTTPS redirect on CloudFront
- Set security headers via CloudFront response headers policy:
  - `Content-Security-Policy`
  - `X-Frame-Options: DENY`
  - `X-Content-Type-Options: nosniff`
  - `Referrer-Policy: strict-origin-when-cross-origin`
- Never embed secrets in frontend code (visible in the browser)

### Vite Specifics
- Use `import.meta.env.VITE_*` for environment variables (not `process.env`)
- Run `npm run build` locally before deploying to catch build errors early

### Development Workflow
- `npm run dev` — Vite dev server with HMR
- `npm run build` — Production bundle to `dist/`
- `npm run preview` — Preview production build locally

### Cost
- S3 + CloudFront: free tier covers most development use
