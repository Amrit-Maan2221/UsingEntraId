### 🔹 PHASE 1 — Azure Entra Preparation

1. Register **Backend API**
2. Expose API scope (`workshop.full_access`)
3. Register **Frontend SPA**
4. Configure SPA redirect URI
5. Grant API permission (Frontend → Backend)
6. Grant admin consent

---

### 🔹 PHASE 2 — Project Setup

7. Install auth libraries
   ```npm install @azure/msal-browser @azure/msal-react```
9. Create environment variables
```
# Create .env in project root folder
VITE_AZURE_TENANT_ID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
VITE_AZURE_CLIENT_ID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
VITE_API_SCOPE=api://xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/workshop.full_access
VITE_API_BASE_URL=https://localhost:5001
```
Perfect. I’ll explain this **like a senior dev onboarding a junior dev on a real project**, using **user-story thinking**, **mental models**, and **clear ownership of files**.
No rushing. No magic.

We’ll cover **only these 3 steps**:

1️⃣ Install auth libraries
2️⃣ Create environment variables
3️⃣ Create MSAL configuration file

Your **Vite + React (JS)** structure is already solid 👍 — we’ll respect it.

---

# 🧩 USER STORY (Big Picture)

> **As a user**,
> I want to sign in using my company Microsoft account,
> so that the frontend can securely talk to our backend API.

To make this happen, **our frontend must**:

* know *who* the user is
* know *where* Azure Entra lives
* know *what permissions* it is requesting
* do this **without hardcoding secrets**

---

# 🔹 STEP 1 — Install Auth Libraries

## 🧠 Senior-level reasoning

We don’t implement OAuth ourselves.
We delegate this to **Microsoft-supported libraries**.

### Libraries we need:

| Library               | Responsibility                       |
| --------------------- | ------------------------------------ |
| `@azure/msal-browser` | Core OAuth logic (tokens, redirects) |
| `@azure/msal-react`   | React integration (context, hooks)   |

These are:

* officially supported
* production-safe
* future-proof

---

## 📌 What problem this step solves

> “I need a reliable way to authenticate users and obtain access tokens.”

Without this:

* You’d manually handle redirects
* Manually store tokens
* Manually refresh tokens

❌ Very dangerous
❌ Very error-prone

---

## ✅ Action (terminal)

```bash
npm install @azure/msal-browser @azure/msal-react
```

Nothing else yet.

---

## 🔹 STEP 2 — Create Environment Variables

## 🧠 Senior-level reasoning

We **never hardcode**:

* client IDs
* tenant IDs
* API identifiers

Because:

* environments change (local, staging, prod)
* secrets leak into git
* builds break across teams

So we use **Vite environment variables**.

---

## 📌 What problem this step solves

> “How does the frontend know *which Azure app* it belongs to?”

Answer: via **environment configuration**, not code.

---

## 📁 Where this belongs

At project root:

```
.env.local
```

(Vite automatically reads it)

---

## ✅ Create `.env.local`

```env
VITE_AZURE_TENANT_ID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
VITE_AZURE_CLIENT_ID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
VITE_API_SCOPE=api://xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/workshop.full_access
VITE_API_BASE_URL=https://localhost:5001
```

---

## 🧠 Important mental notes (as a junior)

* `VITE_` prefix is **mandatory**
* These values are **not secrets**
* Client ID ≠ client secret
* Safe to expose in SPA

If someone steals this → **they still cannot log in without user credentials**

---

## 🔹 STEP 3 — Create MSAL Configuration File

This is the **brain of authentication**.

---

## 🧠 Senior-level reasoning

We want:

* **one place** defining how auth works
* no duplication
* easy debugging
* future extensibility (multi-tenant, B2B)

So we isolate MSAL config into its own file.

---

## 📁 Where this file goes (important)

Based on your structure:

```
src/
└── utils/
    └── msalConfig.js
```

Why `utils`?

* It’s pure configuration
* No React dependency
* Reusable across providers, services, tests

---

## ✅ Create `src/utils/msalConfig.js`

```js
import { PublicClientApplication } from "@azure/msal-browser";

export const msalConfig = {
  auth: {
    clientId: import.meta.env.VITE_AZURE_CLIENT_ID,
    authority: `https://login.microsoftonline.com/${import.meta.env.VITE_AZURE_TENANT_ID}`,
    redirectUri: window.location.origin,
  },
  cache: {
    cacheLocation: "localStorage",
    storeAuthStateInCookie: false,
  },
};

export const msalInstance = new PublicClientApplication(msalConfig);
```

---

## 🧠 Explain this like you’re 1 year in

### `clientId`

👉 Identifies **which frontend app** this is

### `authority`

👉 Identifies **which tenant** users belong to

### `redirectUri`

👉 Where Azure sends the user back after login
(`window.location.origin` = works for dev & prod)

### `cacheLocation`

👉 Keeps login across refresh
(localStorage = recommended for SPA)

---

## 🧩 What we have achieved so far

✅ Auth libraries installed
✅ Environment safely configured
✅ Authentication brain initialized

But:

* ❌ No UI yet
* ❌ No login button
* ❌ No route protection
* ❌ No API calls

That’s expected.

---

### 🔹 PHASE 3 — Auth Infrastructure

Great question. This is exactly the **right moment** to ask it.

I’ll answer it **like a senior dev deciding architecture**, then we’ll **do the steps** cleanly.

---

## 🔍 First — Redux or Context?

### ✅ Decision: **USE CONTEXT (NOT REDUX)**

### ❌ Why NOT Redux for auth?

Auth state:

* `isAuthenticated`
* `account`
* `login() / logout()`

This state is:

* global
* low-frequency changes
* not business data
* not cacheable
* not derived

Redux is best for:

* server data
* complex updates
* normalization
* optimistic updates

Auth is **infrastructure**, not app data.

> **Senior rule:**
> *If Redux DevTools don’t add value → don’t use Redux.*

---

### ✅ Why Context is correct here

| Requirement               | Context |
| ------------------------- | ------- |
| App-wide access           | ✅       |
| Simple API                | ✅       |
| MSAL already uses context | ✅       |
| No reducers needed        | ✅       |
| Less boilerplate          | ✅       |

🧠 **Industry standard**
Microsoft themselves recommend **Context for MSAL**.

---

## ✅ We proceed with CONTEXT

Now let’s do steps **10–13** properly.

---

# 🔹 STEP 10 — Initialize MSAL instance

You already partially did this — we finalize it.

📁 `src/utils/msalConfig.js`
(you already have this — confirm it exists)

```js
import { PublicClientApplication } from "@azure/msal-browser";

export const msalInstance = new PublicClientApplication({
  auth: {
    clientId: import.meta.env.VITE_AZURE_CLIENT_ID,
    authority: `https://login.microsoftonline.com/${import.meta.env.VITE_AZURE_TENANT_ID}`,
    redirectUri: window.location.origin,
  },
  cache: {
    cacheLocation: "localStorage",
  },
});
```

✅ This is the **single MSAL instance** for the app.

---

# 🔹 STEP 11 — Wrap App with MSAL Provider

This makes MSAL available **everywhere**.

---

## 📁 Where?

You already have:

```
src/
└── providers/
    └── AppProviders.jsx
```

Perfect place.

---

## 📄 `src/providers/AppProviders.jsx`

```jsx
import { MsalProvider } from "@azure/msal-react";
import { msalInstance } from "@/utils/msalConfig";

export default function AppProviders({ children }) {
  return (
    <MsalProvider instance={msalInstance}>
      {children}
    </MsalProvider>
  );
}
```

---

## 🧠 Why here?

Because:

* Providers are **infrastructure**
* App.jsx stays clean
* Easy to add more providers later (Theme, Query, etc.)

---

## 🔹 STEP 12 — Create Auth Context Provider

This is **your abstraction over MSAL**.

👉 Components should not talk to MSAL directly.

---

## 📁 Where it belongs

```
src/
└── hooks/
    └── useAuth.jsx
```

Yes — this is both **context + hook** combined (best practice).

---

## 📄 `src/hooks/useAuth.jsx`

```jsx
import { createContext, useContext } from "react";
import { useMsal } from "@azure/msal-react";

const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const { instance, accounts } = useMsal();
  const account = accounts[0];

  const login = () => instance.loginRedirect();
  const logout = () => instance.logoutRedirect();

  return (
    <AuthContext.Provider
      value={{
        account,
        isAuthenticated: !!account,
        login,
        logout,
      }}
    >
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  return useContext(AuthContext);
}
```

---

## 🔹 STEP 13 — Expose Auth Provider in AppProviders

Now we wire **MSAL → AuthContext → App**

---

## 📄 Update `src/providers/AppProviders.jsx`

```jsx
import { MsalProvider } from "@azure/msal-react";
import { msalInstance } from "@/utils/msalConfig";
import { AuthProvider } from "@/hooks/useAuth";

export default function AppProviders({ children }) {
  return (
    <MsalProvider instance={msalInstance}>
      <AuthProvider>
        {children}
      </AuthProvider>
    </MsalProvider>
  );
}
```

---

## 🧠 What your app can now do

Anywhere in the app:

```js
const { isAuthenticated, login, logout, account } = useAuth();
```

✔ Know login state
✔ Trigger login
✔ Trigger logout
✔ Read user info

---

## 🧩 Architecture checkpoint (important)

At this point:

| Layer            | Status       |
| ---------------- | ------------ |
| Azure Entra      | ✅ Configured |
| MSAL instance    | ✅            |
| MSAL Provider    | ✅            |
| Auth Context     | ✅            |
| UI login         | ❌            |
| Route protection | ❌            |
| API calls        | ❌            |

This is **exactly correct** for this stage.

---

### 🔹 PHASE 4 — Routing & Protection

Perfect. This is **real-world routing**, not a toy example 👍
I’ll do this **as a senior dev guiding a junior dev**, respecting **your existing router config, layouts, and pages**.

We’ll implement **steps 14–16** with **minimal changes**, not a rewrite.

---

# 🧠 First: How we think about this (important)

You already have:

* `AuthLayout` → public auth pages (`/auth/*`)
* `MainLayout` → main application shell
* `routerConfig` → route source of truth

So our **job is NOT**:
❌ rewrite routes
❌ move pages
❌ break layouts

Our job **IS**:
✅ block access to `MainLayout` if not logged in
✅ redirect to `/auth/login`
✅ keep lazy loading & Suspense intact

---

# 🔹 STEP 14 — Create Protected Route Component

### 🧠 Senior mental model

A **ProtectedRoute** is NOT a page.
It’s a **gate**.

It decides:

> “Can the user see what’s behind me?”

So it must:

* read auth state
* redirect if unauthenticated
* render children if allowed

---

## 📁 Where it belongs (important)

This is routing logic → put it here:

```
src/routes/ProtectedRoute.jsx
```

---

## 📄 `src/routes/ProtectedRoute.jsx`

```jsx
import { Navigate, Outlet } from "react-router-dom";
import { useAuth } from "@/hooks/useAuth";

export default function ProtectedRoute() {
  const { isAuthenticated } = useAuth();

  if (!isAuthenticated) {
    return <Navigate to="/auth/login" replace />;
  }

  return <Outlet />;
}
```

---

### 🧠 Why `Outlet`?

Because:

* You’re protecting **route groups**, not single pages
* React Router v6 best practice
* Works perfectly with layouts

---

# 🔹 STEP 15 — Add Login Route (You already did ✔️)

Good news:
👉 **You do NOT need to change your login route**

This is already correct:

```js
{
  path: "/auth",
  element: <AuthLayout />,
  children: [
    { path: "login", element: <LoginPage /> },
    { path: "forgot-password", element: <ForgotPasswordPage /> }
  ]
}
```

✅ Login remains public
✅ AuthLayout remains clean
✅ No auth logic inside LoginPage (important)

---

# 🔹 STEP 16 — Protect Main App Routes (key change)

This is where we **slightly tweak** your `routerConfig`.

---

## 🧠 Senior strategy

We wrap **only the protected area** (`/`)
NOT individual pages.

So:

* `/auth/*` → public
* `/` → protected

---

## 📄 Update `routes.config.js`

### 🔴 BEFORE

```js
{
  path: '/',
  element: <MainLayout />,
  children: [
    {
      index: true,
      element: <DashboardPage />,
    },
  ],
}
```

---

### ✅ AFTER (correct, minimal change)

```js
import ProtectedRoute from "./ProtectedRoute";

export const routerConfig = [
  {
    path: "/auth",
    element: <AuthLayout />,
    errorElement: <ErrorPage />,
    children: [
      { path: "login", element: <LoginPage /> },
      { path: "forgot-password", element: <ForgotPasswordPage /> },
    ],
  },

  // 🔐 PROTECTED AREA
  {
    element: <ProtectedRoute />, // 👈 gate
    children: [
      {
        path: '/',
        element: <MainLayout />,
        errorElement: <ErrorPage />,
        children: [
          {
            index: true,
            element: <DashboardPage />,
            handle: { title: 'Dashboard' },
          },
        ],
      },
    ],
  },
];
```

---

## 🧠 Why this structure is correct

| Concern       | Result      |
| ------------- | ----------- |
| Auth logic    | Centralized |
| Layout logic  | Untouched   |
| Pages         | Untouched   |
| Lazy loading  | Still works |
| Redirect loop | Avoided     |
| Future routes | Easy to add |

---

# 🧪 What happens now (flow)

### Scenario 1: Not logged in

```
User → /
ProtectedRoute → ❌
Redirect → /auth/login
```

### Scenario 2: Logged in

```
User → /
ProtectedRoute → ✅
MainLayout → Dashboard
```

---

# 🔹 Login Page — One required tweak (important)

Your **email/password form is now obsolete**
because Azure Entra handles auth.

### ❌ This will NOT be used:

```js
handleSubmit()
email/password inputs
```

---

## ✅ Replace Sign In button behavior

In `LoginPage.jsx`:

```jsx
import { useAuth } from "@/hooks/useAuth";

export default function LoginPage() {
  const { login } = useAuth();

  return (
    <Button className="w-full h-11" onClick={login}>
      Sign in with Microsoft
    </Button>
  );
}
```

✔ UI stays
✔ Auth handled by Entra
✔ No password handling (security win)

---

# 🧩 Checkpoint — what we achieved

✅ Protected routing
✅ Public auth pages
✅ Clean separation of concerns
✅ Layouts untouched
✅ Enterprise-grade routing

At this point:

* App **cannot be accessed without login**
* But API calls are not wired yet


---

### 🔹 PHASE 5 — Token Management


### What we already have

* User can **sign in with Microsoft**
* MSAL stores **ID token + account**
* App knows **who the user is**

### What we DON’T have yet

* A **usable Access Token** for your **Backend API**
* A way to **silently refresh tokens**
* A way to survive **token expiry without logging out**

That’s exactly what steps **17–19** solve.

---

# 🔹 STEP 17 — Configure Token Request Scopes

### 🎯 User Story

> “As a frontend app, when I call the backend API,
> I want Azure to issue me a token that proves
> I’m allowed to call *this* API.”

---

## 🧩 Mental Model

* **Login** ≠ **API Access**
* Login gives you **ID token**
* API access requires **Access token with scopes**

Your backend expects:

```txt
scp = workshop.full_access
```

So frontend must **explicitly request it**.

---

## ✅ Where scopes live (Frontend)

Create a single source of truth.

### `src/auth/authScopes.js`

```js
export const apiScopes = [
  "api://<BACKEND_CLIENT_ID>/workshop.full_access"
];
```

⚠️ IMPORTANT
This must match **exactly** what you defined in:

> Backend App → Expose an API → Scopes

---

## ✅ Update MSAL config

### `src/auth/authConfig.js`

```js
import { PublicClientApplication } from "@azure/msal-browser";

export const msalConfig = {
  auth: {
    clientId: import.meta.env.VITE_AZURE_CLIENT_ID,
    authority: `https://login.microsoftonline.com/${import.meta.env.VITE_AZURE_TENANT_ID}`,
    redirectUri: "/",
  },
  cache: {
    cacheLocation: "localStorage",
    storeAuthStateInCookie: false,
  },
};

export const msalInstance = new PublicClientApplication(msalConfig);
```

Scopes are **not** here — they’re requested per call (best practice).

---

# 🔹 STEP 18 — Implement Silent Token Acquisition

### 🎯 User Story

> “If I’m already logged in,
> don’t ask me to login again —
> just get me a token quietly.”

---

## 🧩 Mental Model

MSAL flow:

1. Try **silent token**
2. If expired → refresh automatically
3. If refresh fails → redirect login

You **never manually refresh tokens**.

---

## ✅ Extend `AuthProvider`

### `src/hooks/useAuth.js`

```js
import { createContext, useContext } from "react";
import { useMsal } from "@azure/msal-react";
import { apiScopes } from "@/auth/authScopes";

const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const { instance, accounts, inProgress } = useMsal();
  const account = accounts[0];

  const login = () =>
    instance.loginRedirect({
      scopes: apiScopes,
    });

  const logout = () => instance.logoutRedirect();

  const getAccessToken = async () => {
    if (!account) return null;

    try {
      const response = await instance.acquireTokenSilent({
        account,
        scopes: apiScopes,
      });

      return response.accessToken;
    } catch (error) {
      // fallback if silent fails
      await instance.acquireTokenRedirect({
        scopes: apiScopes,
      });
    }
  };

  return (
    <AuthContext.Provider
      value={{
        account,
        isAuthenticated: !!account,
        inProgress,
        login,
        logout,
        getAccessToken,
      }}
    >
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  return useContext(AuthContext);
}
```

---

## 🧠 What you just achieved

* Tokens requested **only when needed**
* Silent refresh handled by MSAL
* Backend always receives **valid tokens**

---

# 🔹 STEP 19 — Handle Token Expiration (Axios Layer)

### 🎯 User Story

> “Every API call should automatically attach a token,
> and if the token expires, the app should recover gracefully.”

---

## 🧩 Mental Model

* **Components should NOT know about tokens**
* Axios should:

  * attach token
  * retry silently
  * redirect only if needed

---

## ✅ Create Axios Instance

### `src/services/apiClient.js`

```js
import axios from "axios";
import { msalInstance } from "@/auth/authConfig";
import { apiScopes } from "@/auth/authScopes";

const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
});

apiClient.interceptors.request.use(async (config) => {
  const accounts = msalInstance.getAllAccounts();
  if (accounts.length === 0) return config;

  const response = await msalInstance.acquireTokenSilent({
    account: accounts[0],
    scopes: apiScopes,
  });

  config.headers.Authorization = `Bearer ${response.accessToken}`;
  return config;
});

export default apiClient;
```

---

## ✅ Usage in Services

### `src/services/profile.service.js`

```js
import apiClient from "./apiClient";

export const getProfile = () => {
  return apiClient.get("/api/profile");
};
```

✔ Components stay clean
✔ Tokens handled centrally
✔ Expiry auto-managed

---

# 🧠 Final Architecture (Clean & Scalable)

```
React UI
  ↓
AuthProvider (login, logout, token)
  ↓
Axios Interceptor
  ↓
Backend API (scope-protected)
```

---

# ❓ Common Junior Dev Confusion (Answered)

### ❌ Do I manually refresh tokens?

No. **MSAL does it for you**.

### ❌ Do I store tokens in Redux?

No. MSAL cache is enough.

### ❌ Do I protect every API with policy?

Yes — backend remains secure even if frontend is bypassed.


---

### 🔹 PHASE 6 — API Communication

20. Create Axios instance
21. Add request interceptor
22. Attach access token automatically

---

### 🔹 PHASE 7 — UI Integration

23. Show login button
24. Show logout button
25. Display logged-in user info
26. Hide UI when not authenticated

---

### 🔹 PHASE 8 — Authorization (Client-side)

27. Read roles / claims from token
28. Gate routes by role
29. Gate sidebar menu items
30. Handle forbidden actions gracefully

---

### 🔹 PHASE 9 — Error Handling

31. Handle 401 globally
32. Handle 403 globally
33. Redirect to login on auth failure

---

### 🔹 PHASE 10 — Production Hardening

34. Disable implicit flows
35. Lock redirect URIs
36. Enable HTTPS only
37. Validate token audience & issuer

---

### 🔹 PHASE 11 — Future Enhancements

38. Add refresh-safe silent login
39. Multi-tenant support (optional)
40. External user access (B2B)

---

If you want, next we can:

* Start **PHASE 2 step-by-step**
* Or jump directly to **PHASE 6 (Axios)**
* Or align this roadmap with your **sidebar permissions**

Just say **which phase to start** 🚀
