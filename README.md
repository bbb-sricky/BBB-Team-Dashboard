# BBB Team Dashboard
**BITbyBIT · Azure Static Web App + Microsoft Graph (Outlook) + Cosmos DB**

A single-page team dashboard that connects to your Microsoft 365 account to show your Outlook inbox, upcoming calendar events, send emails, and create calendar entries — all logged to Azure Cosmos DB.

---

## Project Structure

```
bbb-dashboard/
├── public/
│   ├── index.html                  ← The entire app (single file)
│   └── staticwebapp.config.json    ← Azure SWA routing config
├── .github/
│   └── workflows/
│       └── deploy.yml              ← GitHub Actions auto-deploy
└── README.md
```

---

## Step 1 — Push to GitHub

```bash
# In this folder:
git init
git add .
git commit -m "Initial commit — BBB Team Dashboard"

# Create a new repo on github.com, then:
git remote add origin https://github.com/YOUR_USERNAME/bbb-dashboard.git
git branch -M main
git push -u origin main
```

---

## Step 2 — Create Azure Static Web App

1. Go to [portal.azure.com](https://portal.azure.com)
2. Search for **Static Web Apps** → **Create**
3. Fill in:
   - **Subscription**: your subscription
   - **Resource Group**: `rg-bbb-dashboard` (create new)
   - **Name**: `bbb-team-dashboard`
   - **Plan type**: Free
   - **Region**: East Asia (closest to Jakarta)
   - **Deployment source**: GitHub
4. Click **Sign in with GitHub** → authorize → select your repo and `main` branch
5. **Build Details**:
   - Build Preset: `Custom`
   - App location: `/public`
   - Api location: *(leave blank)*
   - Output location: *(leave blank)*
6. Click **Review + Create** → **Create**

Azure will automatically add the `AZURE_STATIC_WEB_APPS_API_TOKEN` secret to your GitHub repo and trigger the first deployment.

> Your app URL will be something like: `https://polite-wave-abc123.azurestaticapps.net`

---

## Step 3 — Azure App Registration (for real Outlook login)

1. Go to **Azure Active Directory** → **App registrations** → **New registration**
2. Name: `BBB Team Dashboard`
3. Supported account types: **Accounts in any organizational directory (Any Azure AD directory)**
4. Redirect URI:
   - Type: **Single-page application (SPA)**
   - URL: `https://YOUR-APP.azurestaticapps.net` *(use your actual URL from Step 2)*
   - Also add `http://localhost:3000` for local testing
5. Click **Register**
6. Copy the **Application (client) ID** and **Directory (tenant) ID**

### Add API Permissions
Under your new app → **API permissions** → **Add a permission** → **Microsoft Graph** → **Delegated**:

| Permission | Purpose |
|---|---|
| `User.Read` | Get signed-in user's profile |
| `Mail.ReadWrite` | Read inbox emails |
| `Mail.Send` | Send emails |
| `Calendars.ReadWrite` | Read and create calendar events |

Click **Grant admin consent** (if you're the Azure AD admin).

---

## Step 4 — Enter Client ID in the App

1. Open your deployed app URL
2. On the login screen, paste:
   - **Client ID** → Application (client) ID from Step 3
   - **Tenant ID** → Directory (tenant) ID (or type `common` for any org)
3. Click **Sign in with Microsoft**

The app remembers your Client ID in `localStorage` — you only need to enter it once per browser.

---

## Step 5 — Connect Cosmos DB (optional, for persistent logging)

The current app simulates Cosmos DB using `localStorage`. To connect a real Cosmos DB:

1. Create a **Cosmos DB account** in Azure (API: NoSQL Core)
2. Create database: `bbb-dashboard-db`, container: `activityLog`, partition key: `/userId`
3. Create an **Azure Function** (HTTP trigger) as a proxy — the app calls the function, the function writes to Cosmos DB using the SDK
4. Add the Function URL to the app's config

> A full Azure Functions backend guide is available — ask for it separately.

---

## Local Development

No build tools needed — just open `public/index.html` in your browser.

```bash
# Optional: use a local server (avoids MSAL redirect issues)
npx serve public
# then open http://localhost:3000
```

---

## Features

- ✅ Microsoft 365 sign-in via MSAL.js (OAuth2 / Azure AD)
- ✅ Read Outlook inbox (last 20 emails)
- ✅ Compose & send emails
- ✅ View upcoming 7-day calendar
- ✅ Create calendar events with attendees
- ✅ Cosmos DB activity log (simulated, upgradeable to real)
- ✅ Dark mode (persisted)
- ✅ Demo mode (no Azure account needed to preview)
- ✅ Auto-deploy on `git push` via GitHub Actions

---

## Tech Stack

| Layer | Technology |
|---|---|
| Hosting | Azure Static Web Apps (Free tier) |
| Auth | MSAL.js 2.x (OAuth2 PKCE) |
| Email + Calendar | Microsoft Graph API v1.0 |
| Activity Log | Azure Cosmos DB (NoSQL Core) |
| CI/CD | GitHub Actions |
| Framework | Vanilla HTML/JS — zero build step |

---

*BITbyBIT · tech savvy. business smart.*
