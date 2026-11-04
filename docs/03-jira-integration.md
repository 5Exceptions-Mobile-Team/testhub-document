# 3. Connect TestHub to Jira

This guide explains how a team gets TestHub inside **their** Jira site, and how the connection works. It has three parts:

- **Part A:** what the team and the TestHub admin must agree and prepare.
- **Part B:** what the Jira site admin does (install the app).
- **Part C:** what the TestHub admin does (allow the site and project).

You need all three. The TestHub page shows data only after Parts B and C are done.

## How the connection works

```
 Jira user opens "TestHub" in the project sidebar
        │   (Atlassian signs the request)
        ▼
 TestHub verifies: signature, app ID, Jira site (cloud ID), Jira project is allowed
        ▼
 TestHub creates or finds the user's TestHub account and answers
```

- The app is called **TestHub** and is an Atlassian **Forge** app. It adds a **TestHub** page to every Jira project's sidebar, a **Tests** panel on issues, and an optional Confluence macro.
- Nothing is copied to Atlassian. The app is a thin window onto the TestHub server.
- Access is controlled by TestHub, not by Jira alone: a site must be on TestHub's allowed list, and a Jira project must be on its allowed projects list.

## Part A: before you start

Collect these:

| What | Example | Who knows it |
|---|---|---|
| Jira site address | `yourteam.atlassian.net` | The team |
| Jira project key to use | `TA` | The team |
| A Jira **site admin** (not just a project admin) | | The team |
| A **TestHub admin** | | 5Exceptions TestHub owners |
| The TestHub install link | signed link from the app owner | The TestHub owners |

A TestHub project must exist for the team, with its **Jira project key** set to the Jira project key (Settings → Project in TestHub). TestHub opens that project automatically when the page is opened from that Jira project.

## Part B: the Jira site admin installs the app

Only a site admin can install an app. A project admin cannot.

1. Ask the TestHub owners for the **install link** for the TestHub app. They generate it in the Atlassian developer console (the app's **Distribution** page, with **Jira** selected). The link is signed and tied to the app, so share it only with the site admin.
2. The site admin opens the link while signed in to **their own** Jira site and picks the site in the install dialog. If the dialog shows "No options", the browser is signed in to another site, so switch accounts or use a private window.
3. Accept the permissions screen. The app asks for no Atlassian scopes: it reads nothing from Jira by itself.
4. Wait a minute, then open a project in that Jira site. **TestHub** now appears in the left sidebar.

Notes:
- There can be only one installation of the app per site. Installing again says "already installed". A development installation of the app on the same site blocks the production one; uninstall it first (`forge uninstall -i <id> -e development`).
- Optional Confluence: the same app has a **TestHub results** macro. Ask the site admin to install it on Confluence too, then type `/TestHub` on a page.
- To remove the app: Jira settings → Apps → Manage apps → TestHub → Uninstall.

## Part C: the TestHub admin allows the site and project

The TestHub admin signs in to the **web app** and opens **Settings → Jira app**.

1. **Allowed Jira site IDs.** Add the team's site ID (its cloud ID). To find it, open `https://<site>.atlassian.net/_edge/tenant_info` in a browser; the answer contains `"cloudId"`. The **Detect** button on this page works only for the site saved under Settings → Jira.
2. **Jira projects where TestHub is enabled.** Add the team's Jira project key (for example `TA`). Other projects see "TestHub is not enabled" and no data. An empty list means nobody.
3. **Role for people who open TestHub inside Jira.** New users start as **Tester** or **Viewer**. Promote individuals later in Settings → Users (marked "Jira").
4. **Allow "Create test" from the Jira panel** (optional). If on, anyone who can view an issue can create a draft test from the issue panel.
5. Click **Save**, then **Check connection to Atlassian** (it should report that it reached Atlassian's signing keys).
6. Make sure a TestHub **project** exists with the same Jira key (Settings → Project), or create it from the project picker.

The app ID and public address on this page are set once for the platform and do not change per team.

## What the team sees

- **TestHub** in the Jira project sidebar: the full application (Dashboard, Library, Cycles, Plans, Reports, Settings).
- **Tests** panel on every issue (use "More" if hidden): the tests that cover the issue and their last results.
- Each person is created in TestHub the first time they open the page (named like "Jira user 4f2a1c" until the Jira connection can read their name) with the default role.

## Jira issue status, search and defects (the Jira connection)

The page works inside Jira without any token. To show **live Jira issue status**, **search Jira issues**, **create defects** from failed tests and add comments, TestHub also needs a Jira API connection:

1. A TestHub admin opens **Settings → Jira** in the web app.
2. Enter the Jira site address, the email of a Jira account that can browse and create issues in the project, and an API token (id.atlassian.com → Security → API tokens).
3. Set the default Jira project key, choose whether failed tests automatically create a Jira issue, and click **Save & test connection**.

**Limit:** TestHub holds **one** Jira connection. Issue status, search and defects work only for that site. A second team on a different Jira site can use the TestHub page and the Tests panel, but linked-issue status, Jira search and defect creation will look at the connected site, not theirs. A TestHub with a separate connection per site is not built.

## Updating the app (for the TestHub owners)

User interface changes are delivered by deploying the Forge app; no reinstall is needed when the permissions and manifest do not change.

```bash
npm run build:jira-app          # builds the app screens
cd jira-app
forge lint
forge deploy --non-interactive -e production
```

If a change adds permissions or modules, `forge deploy` still works but each site admin must **upgrade** (`forge install --upgrade`, or accept the prompt in Jira). After a deploy, ask users to hard-refresh the TestHub page.

## Troubleshooting

| What you see | Likely cause and fix |
|---|---|
| TestHub is not in the sidebar | The app is not installed on this site (Part B), or the install is still finishing. |
| "This Jira site is not allowed to use this TestHub" | The site ID is missing from Settings → Jira app (Part C step 1). |
| "TestHub is not enabled for the Jira project X" | Add the key under "Jira projects where TestHub is enabled" (Part C step 2). |
| "Internal error" | The TestHub server is unreachable, or the app still points at an old address. The TestHub owners should check the server and the app's address. |
| Page is blank for about 30 seconds | The database was asleep. Wait and reload. |
| "Jira is not configured" when searching or creating a defect | Settings → Jira is empty (see above). |
| "Jira returned 404 for ..." | The Jira connection points at a site where the project or issue does not exist, or the site address is wrong. Press Save & test connection and check the project key. |
| You are "tester" but need more | A TestHub admin promotes you in Settings → Users. |
