You have:

* **Backend API** → ASP.NET Core (secured API)
* **Frontend Web App** → React + Vite + Shadcn UI
* **Identity Provider** → **Azure Entra ID** (formerly Azure AD)

🎯 **Goal**

* User signs in via Entra ID from React
* React gets an **access token**
* React calls **Backend API**
* Backend **validates token**, **authorizes by scopes/roles**

---

# 🧭 Big Picture (Mental Model)

```
[ React App ] ──(login)──▶ [ Azure Entra ID ]
     │                         │
     │   access_token          │
     └──────▶ [ Backend API ] ◀┘
                 (JWT validation)
```

We will do this in **7 clear steps**:

1️⃣ Entra ID App Registration – Backend API
2️⃣ Expose API (Scopes / App ID URI)
3️⃣ Entra ID App Registration – Frontend (SPA)
4️⃣ Grant API permissions (Frontend → Backend)
5️⃣ Backend API configuration (ASP.NET Core)
6️⃣ Frontend configuration (React + MSAL)
7️⃣ Authorization (Scopes / Roles)

We will **stop after each step** and validate.

---

## 🔹 STEP 1 — Register **Backend API** in Azure Entra ID

This app represents **your secured API**.

### 1. Go to Azure Portal

```
Azure Portal → Microsoft Entra ID → App registrations → New registration
```

### 2. Create the App

Fill like this:

* **Name**:

  ```
  WorkshopSaaS-Backend-API
  ```

* **Supported account types**:
  ✅ *Accounts in this organizational directory only*
  (Single-tenant – best for internal SaaS)

* **Redirect URI**:
  ❌ Leave empty (API does NOT need redirect)

👉 Click **Register**

---

### 3. Note these values (VERY IMPORTANT)

From **Overview** page:

* 📌 **Application (client) ID**
* 📌 **Directory (tenant) ID**

Save them somewhere:

```
Backend API
- ClientId = xxxxx
- TenantId = xxxxx
```

⚠️ Do NOT create secrets for API — **not needed**

---

## 🔹 STEP 2 — Expose the Backend API (Scopes)

Now we tell Entra ID:

> “This API can be called by other apps”

### 1. Go to:

```
App registrations → WorkshopSaaS-Backend-API → Expose an API
```

### 2. Set Application ID URI

Click **Set** and use:

```
api://<Backend-ClientId>
```

Example:

```
api://3c1f9c3e-xxxx-xxxx-xxxx-xxxx
```

Click **Save**

---

### 3. Create API Scope

Click **Add a scope**

Fill like this:

* **Scope name**

  ```
  workshop.full_access
  ```

* **Who can consent**

  ```
  Admins
  ```

* **Admin consent display name**

  ```
  Access WorkshopSaaS API
  ```

* **Admin consent description**

  ```
  Allows the app to access WorkshopSaaS backend API on behalf of the signed-in user.
  ```

* **State** → Enabled ✅

Click **Add scope**

✔️ Backend API is now protected by **OAuth scopes**

---

## 🔹 STEP 3 — Register **Frontend React SPA**

This app represents **your React UI**.

### 1. New App Registration

```
App registrations → New registration
```

### 2. Fill details

* **Name**

  ```
  WorkshopSaaS-Frontend
  ```

* **Supported account types**

  ```
  Single tenant
  ```

* **Redirect URI**

  * Platform: **Single-page application (SPA)**
  * URI (Vite default):

    ```
    http://localhost:5173
    ```

👉 Click **Register**

---

### 3. Save these values

From Overview:

```
Frontend App
- ClientId
- TenantId
```

---

## 🔹 STEP 4 — Configure Frontend as SPA

Go to:

```
WorkshopSaaS-Frontend → Authentication
```

✔️ Make sure:

* **Platform** = SPA
* Redirect URI exists:

  ```
  http://localhost:5173
  ```

Enable:

* ✅ **Access tokens**
* ✅ **ID tokens**

❌ No client secret needed (SPA rule)

---

## 🔹 STEP 5 — Grant API Permission (Frontend → Backend)

This is where many people mess up — we won’t 🙂

### 1. Go to:

```
WorkshopSaaS-Frontend → API permissions → Add a permission
```

### 2. Select:

```
My APIs → WorkshopSaaS-Backend-API
```

### 3. Choose:

```
Delegated permissions
```

✔️ Select:

```
access_as_user
```

Click **Add permissions**

---

### 4. Grant Admin Consent

Click:

```
Grant admin consent for <Tenant>
```

✅ Status should turn **green**

---

## ✅ CHECKPOINT (VERY IMPORTANT)

At this point:

* ✔️ Backend API registered
* ✔️ Scope created
* ✔️ Frontend registered as SPA
* ✔️ Frontend can request access token for API

---
### 🔹 STEP 6 — Backend API (.NET)

We will configure:

* `AddMicrosoftIdentityWebApi`
* JWT validation
* Scope authorization

### 🔹 STEP 7 — React App

We will configure:

* `@azure/msal-browser`
* `@azure/msal-react`
* Login button
* Call API with token

---

### 👉 Tell me:

1️⃣ **ASP.NET Core version** (7 / 8 / 9?)
2️⃣ Is your backend **Minimal API or Controllers?**

Then we move to **STEP 6 – Backend API configuration** 🚀
