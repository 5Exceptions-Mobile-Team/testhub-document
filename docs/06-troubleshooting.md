# 6. Troubleshooting and FAQ

## Quick fixes

| Problem | What to do |
|---|---|
| The page is slow or blank for up to 30 seconds | The database sleeps when idle. Wait and reload. It is quick afterwards. |
| After an update the page looks old | Hard-refresh the page (Cmd+Shift+R on Mac, Ctrl+Shift+R on Windows). Inside Jira refresh the whole Jira tab. |
| "An internal error occurred" with a reference | A server problem. Send the reference (`ref`) and what you clicked to the TestHub owners. |
| "Please sign in" | Your session ended. Sign in again (web app) or reload the Jira page. |
| "Only an admin can do that" | The action needs the Admin role. Ask a TestHub admin. |
| "Your account is read-only" | You are a Viewer. Ask an admin to change your role. |

## Jira

| Problem | Cause and fix |
|---|---|
| TestHub is missing from the Jira sidebar | The app is not installed on that site. A **site admin** installs it (guide 3, part B). |
| "This Jira site is not allowed to use this TestHub" | The site's cloud ID is not in Settings → Jira app. |
| "TestHub is not enabled for the Jira project X" | Add X to the enabled Jira projects in Settings → Jira app. |
| "Jira is not configured" | Fill in Settings → Jira (site, email, token) and use Save & test connection. |
| "Jira returned 404 for POST /search/jql" | The saved Jira site address is wrong, or the project key does not exist on that site. |
| Search finds nothing | The issue is in a project on a different Jira site than the one TestHub is connected to, or the account cannot browse it. |
| Install dialog shows "No options" | The browser is signed in to a different Atlassian site. Switch account, or use a private window. |
| "App already installed on this site" | Only one installation per site. Remove the other installation (for example a development one) first. |
| A person appears as "Jira user 4f2a1c" | TestHub could not read their name from Jira. Fix the Jira connection, or an admin can rename them in Settings → Users. |

## Test library

| Problem | Cause and fix |
|---|---|
| Export CSV has too many tests | Export follows the selection: choose a folder, Unfiled or All first. A selected folder includes its subfolders. |
| I cannot find a test | Check the filters (status, priority, automation, component, Generated) and the selected folder. Search also covers step text. |
| A test I archived is gone from the list | Archived tests are Deprecated. Set the status filter to Deprecated. |
| I cannot edit or approve a test | You may be a Viewer, the test may be awaiting review, or you are not an Approver. |
| The page scrolls sideways | Update the page; the table now scrolls inside its own area. |

## API and tools

| Problem | Cause and fix |
|---|---|
| `401 Missing or invalid API key` | The header must be `Authorization: Bearer th_...` and the key must not be revoked. |
| `403 This API key cannot access that project` | The key belongs to another project. Use the key created for this project. |
| `403 Legacy keys support results upload only` | An old key. Create a new project key. |
| `403 API keys can edit only generated Draft tests` | The test is approved, not Draft, or was written by a person. |
| `429` | More than 60 requests a minute on that key. Wait for `Retry-After` seconds. |
| Results do not match tests | Put the test key (`TA-T12`) in the test name, or use an exact name. Use `autoCreate=true` to create missing tests. |
| Generated cases are not in my folder | They go to the **Generated** folder as Draft with the label `generated`. Use the Generated button in the Library. |
| Webhook says "Could not connect" or "not allowed" | The URL must be https and public. Private addresses are refused. |
| Webhook receiver rejects the signature | Verify against the **raw** body, use the secret shown at creation and the `t` value from the header. |

## FAQ

**Can another company or team use our TestHub inside their Jira?** Yes. Their Jira site admin installs the app and a TestHub admin adds their site ID and project key (guide 3). Limits: one Jira connection per TestHub, so linked-issue status, Jira search and defects use the connected site only.

**Does TestHub store my Jira data?** It stores Jira issue **keys** you link to tests, and reads issue status and summaries live from Jira. The Forge app itself reads nothing from Jira.

**Who can see our tests?** People who are signed in to TestHub (web) or whose Jira project is enabled (inside Jira), and API keys for their own project only.

**Can we import from Zephyr, Xray or TestRail?** Yes, through the import wizard in the Test Library (CSV exports). See guide 2, section 8.

**Can reports be emailed?** Not yet. Scheduled reports go to Slack or any URL.

**How do we connect Slack?** Create an incoming webhook in Slack and paste its URL as a scheduled report, or point a TestHub webhook at your own service that forwards events to Slack.

**What happens to old keys?** Keys made before project keys existed (shown as "Legacy") keep working for result uploads only. Replace them with project keys when convenient.

## Getting help

Send the TestHub owners: what you did, what you expected, what you saw (a screenshot helps), the project key, and the `ref` shown with any internal error.
