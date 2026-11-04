# 5. Administrator guide

For TestHub admins. Everything below is in the web app under **Settings** unless it says otherwise. Admin-only settings are hidden from other roles.

## 1. Projects

- Create projects from the project picker. A project has a **name**, a **key** (2 to 10 letters or digits, starting with a letter; it becomes the prefix of every ID) and an optional description.
- **Settings → Project:** edit the name and description, set the **Jira project key** (needed for defects, Jira search defaults and for opening the right project from Jira), and delete the project. Deleting removes all its tests, cycles, plans and results, so it asks you to type the key to confirm.
- **Review workflow** (same page): turn it on to use Draft, In Review, Approved and Deprecated. Options:
  - editing an approved test sends it back to Draft;
  - only approved tests can be added to cycles;
  - authors cannot approve their own submissions.

## 2. Users and roles

**Settings → Users.**

| Role | Rights |
|---|---|
| Admin | Everything, including settings, users, keys, webhooks and reports. |
| Tester | Write tests, run cycles and plans, record results, comment. |
| Viewer | Read only. |

- Mark a user as an **Approver** to let them approve tests in review (admins always can).
- Web app users sign in with email and password (at least 10 characters). Add users with name, email, role and a temporary password, and use **Reset password** when needed. Deleting a user keeps their results.
- People who open TestHub inside Jira are added automatically with the default role from **Settings → Jira app**. They show a "Jira" badge, have no TestHub password and are named after their Jira account. Promote them here.
- There must always be at least one admin. The first admin is created when the platform is first set up.

## 3. Jira connection

**Settings → Jira.** Needed for live issue status, Jira search, creating defects and comments.

1. Enter the Jira site (`https://yourcompany.atlassian.net`), the email of a service or admin account and an API token from id.atlassian.com → Security → API tokens. The account must be able to browse the project and create issues.
2. Set the **default Jira project key**.
3. Options: create a Jira issue automatically when a test fails (a Bug if the project has it, otherwise a Task) and, when a failed test later passes, also move the issue to Done (a comment is always added).
4. Click **Save & test connection**. A success message shows the name of the connected account.

The token is encrypted in the database and never shown again. There is **one** connection per TestHub.

## 4. Jira app (TestHub inside Jira)

**Settings → Jira app.** See [guide 3](03-jira-integration.md) for the full procedure. The fields:

| Field | Purpose |
|---|---|
| Forge app ID | Identifies the TestHub app (set once for the platform). |
| Allowed Jira site IDs | Only these Jira sites may use this TestHub. |
| Public HTTPS address | Used for "Open in TestHub" links in Jira. |
| Jira projects where TestHub is enabled | Other projects see "not enabled". Empty means nobody. |
| Role for people who open TestHub inside Jira | Tester or Viewer. |
| Allow "Create test" from the Jira panel | Lets issue viewers create draft tests. |

**Check connection to Atlassian** verifies that TestHub can reach Atlassian's signing keys.

## 5. Confluence

**Settings → Confluence.** TestHub uses the Jira connection to talk to Confluence. Pick a **default space** for **Publish to Confluence** on cycles and plans. The account needs "Add pages" in that space. Publishing again updates the same page. You can also link Confluence pages to tests, and install the live **TestHub results** macro (guide 3).

## 6. Custom fields

**Settings → Custom fields** (also available inside Jira). Add fields per project for **test cases** or **test cycles**: text, long text, number, dropdown, checkbox or date; mark them required; reorder, rename or delete them. Fields appear in the forms, as Library columns, in reports and in CSV import and export as `Custom: <name>` columns. A field's type cannot be changed after it is created.

## 7. API keys

**Settings → API keys** (inside Jira: **Keys**, limited to the current project). Create one key per tool and project, copy the token when it is shown, and **Revoke** keys that are no longer used. The table shows the project, the first characters of the token, when it was created and when it was last used. See [guide 4](04-connect-apps.md) for what a key allows.

## 8. Webhooks and scheduled reports

**Settings → Webhooks & reports** (web app only).

- **Webhooks:** name, https URL and the events to send. The signing secret is shown once. Use **Send test** to check it, and the Active switch to pause it. The table shows the result of the last delivery.
- **Scheduled reports:** name, URL, daily or weekly, day and hour (UTC). Use **Send now** to test. A Slack incoming-webhook URL works as it is.

Details and the signature check are in [guide 4](04-connect-apps.md).

## 9. Onboarding a new team: checklist

1. Create the TestHub project and set its Jira project key.
2. Add the team's Jira site ID and project key under Settings → Jira app.
3. Ask the team's Jira site admin to install the TestHub app (guide 3, part B).
4. Check that a team member can open **TestHub** in their Jira project; promote the right people to Tester or Admin.
5. Create an API key for their pipeline if they need one.
6. Optional: a webhook for failures and a weekly report to their channel.

## 10. Operations notes (for the platform owners)

- **Hosting:** the application runs on AWS (CloudFront in front of a container service, with a PostgreSQL database). Releases go through the project's build pipeline and are approved by the owner. A release replaces the single running task, so expect about 1.5 minutes of downtime.
- **Database sleep:** the database scales to zero when idle, so the first request after about 15 minutes of inactivity can take up to about 30 seconds. This is a cost choice.
- **Backups and secrets:** database secrets and the encryption key are held in the cloud secret store and are not part of the code. Losing the encryption key means the saved Jira token and webhook secrets must be entered again.
- **Schema changes** are applied automatically at startup, only after an explicit release approval, and are additive. Each release records the rollback steps in the project's database notes.
- **Scheduled reports** run inside the server process every minute; each run is claimed in the database so two servers never send the same report twice.
- **The Jira app** is deployed separately with the Forge command line (guide 3, "Updating the app").
- **Security:** sessions use HTTP-only cookies; passwords are stored hashed; API keys are stored hashed; webhooks refuse private network addresses; every request from Jira is checked for signature, app, site and allowed project.
