---
name: add-auth
description: Adds Cognito authentication and route protection. Use when adding login, signup, authentication, or user management.
---

# Add Auth — Cognito Authentication

You are adding Cognito authentication to protect API routes and manage user accounts.

## Prerequisites
- Lambda handlers exist (from create-api step)
- API Gateway is configured

## What To Do

### Phase 1: Plan

Based on the migration plan, show the user:
- Public routes (no auth needed)
- Protected routes (require login)

Ask the user:
1. "Here are the routes I'll protect — does this look right?"
2. "Do you want email verification for new users? (Recommended)"
3. "Password requirements — min 8 characters, uppercase, lowercase, number, symbol. Sound good?"
4. "Do you want social login (Sign in with Google)? I'd recommend starting without it."
5. "Should users be able to reset their password via email?"

### Phase 2: Execute

1. **Create `infrastructure/lib/auth-stack.ts`**
   - Cognito User Pool with:
     - Email as sign-in alias
     - Email verification enabled (auto-verify)
     - Strong password policy (min 8, mixed case, numbers, symbols)
     - Account recovery via email
     - Self-registration enabled
   - User Pool Client with:
     - No client secret (for browser-based auth)
     - Auth flows: USER_PASSWORD_AUTH, USER_SRP_AUTH
     - Token validity periods
   - Export User Pool ID and Client ID

2. **Add Cognito Authorizer to API Gateway**
   - Update `api-stack.ts` to:
     - Import the User Pool from auth-stack
     - Create a Cognito Authorizer
     - Apply authorizer to protected routes only
     - Leave public routes unprotected

3. **Create React auth integration** (if frontend exists)
   - Install: `cd frontend && npm install amazon-cognito-identity-js`
   - Create `frontend/src/lib/auth.ts` with:
     - `signUp(email, password)`, `confirmSignUp(email, code)`, `signIn(email, password)`
     - `signOut()`, `getSession()`, `getIdToken()`
   - Environment variables: `VITE_COGNITO_USER_POOL_ID`, `VITE_COGNITO_CLIENT_ID`
   - Create `frontend/src/hooks/useAuth.ts` — manages auth state, provides auth functions, checks session on mount
   - Create `frontend/src/context/AuthContext.tsx` — wraps app to provide auth state
   - Create components in `frontend/src/components/`:
     - `SignInForm.tsx`, `SignUpForm.tsx`, `ConfirmSignUp.tsx`, `ProtectedRoute.tsx`

4. **Update API client** to include auth token:
   Update `frontend/src/lib/api.ts`:
   ```typescript
   import { getIdToken } from './auth';

   export async function apiFetch(path: string, options?: RequestInit) {
     const token = await getIdToken();
     const headers: Record<string, string> = {
       'Content-Type': 'application/json',
       ...options?.headers as Record<string, string>,
     };
     if (token) {
       headers['Authorization'] = `Bearer ${token}`;
     }
     const response = await fetch(`${API_BASE}${path}`, { ...options, headers });
     if (!response.ok) throw new Error(`API error: ${response.status}`);
     return response.json();
   }
   ```

5. **Update React Router** to protect routes:
   ```tsx
   <Route path="/dashboard" element={
     <ProtectedRoute>
       <DashboardPage />
     </ProtectedRoute>
   } />
   ```

### Phase 3: Verify

1. Run `cd infrastructure && npx cdk synth` to verify compilation
2. Show the user: Cognito config, protected vs public routes, frontend auth integration
3. Update `.migration/plan.md` to mark add-auth as complete

## Important Notes

### Security
- Cognito SDK stores tokens in localStorage by default — mention this trade-off; for production, discuss httpOnly cookies
- Set reasonable token expiration (1 hour access, 30 days refresh)
- Use SRP auth flow (never send plaintext passwords)
- Mention MFA as an optional enhancement

### Best Practices
- Use email as the primary identifier (not username)
- Enable email verification
- Configure account recovery via email only (not phone, to keep it simple)
