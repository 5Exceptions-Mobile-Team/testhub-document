# 1. Overview

## What TestHub is

TestHub is a self-hosted test management platform, modelled on Zephyr Scale and connected to Jira and Confluence. Teams use it to:

- write and organise test cases in folders,
- group tests into **cycles** (a build, a sprint, a release) and run them,
- link tests to Jira issues and raise Jira defects from failed tests,
- see progress and quality in dashboards and reports,
- receive automated results and generated test cases from other tools.

## Where you use it

| Surface | What it is | Who uses it |
|---|---|---|
| **Web app** | The full application in a browser. | Everyone; admins manage settings here. |
| **TestHub inside Jira** | The same application as a **TestHub** page in every Jira project's sidebar, signed in through Jira. | Testers and developers who already work in Jira. |
| **Jira issue panel** | A **Tests** panel on a Jira issue showing the tests that cover it and their latest results. | Anyone viewing an issue. |
| **Confluence macro** | A live "TestHub results" macro for Confluence pages, and one-click publishing of a cycle or plan to a Confluence page. | Test leads and managers. |
| **API** | Endpoints for CI pipelines and test generators, protected by project API keys. | Developers, automation, tool owners. |
| **Webhooks and scheduled reports** | TestHub can notify other systems and post a summary to Slack or any URL. | Admins, team channels. |

## Key ideas

| Word | Meaning |
|---|---|
| Project | A set of tests, cycles and plans. Its key (for example `TA`) is the prefix of every ID. |
| Test case | One thing to check, written as steps. IDs look like `TA-T12`. |
| Test cycle | A run of selected tests for a build or sprint. IDs look like `TA-R3`. |
| Test plan | A group of cycles whose progress is rolled up. IDs look like `TA-P1`. |
| Execution | The result of one test inside one cycle. |
| Folder | A nested folder for tests (Library) or cycles (Test Cycles). |

## Feature summary

**Test library**
- Nested folders with a ⋯ menu (add subfolder, rename, delete, expand all, collapse all, create a test cycle from the folder, copy ID, copy link), folder search, and drag-and-drop of folders and tests. Counts include subfolders; archived tests have their own view with Restore.
- Search where every word must match, with a switch between names and labels or everything including steps; matches are highlighted. Order by newest, oldest, recently updated or name. Filters for status, priority, automation and component.
- Columns you can show or hide: priority, status, automation, labels, owner, component, estimated minutes, steps, last result, Jira and updated; plus custom fields.
- Bulk actions: clone, archive, move to a folder, set priority, add to a cycle, delete (admins).
- Step-by-step, plain text or BDD (Given/When/Then) scripts; reusable steps (a step can call another test); parameters and datasets (one result per data row).
- Versions: any content change creates a new version, which you can view and restore.
- Review workflow: Draft, In Review, Approved, Deprecated, with approvers and a history.
- Automation status: Automated, Can't automate, Manual (new tests start as Manual). Estimated time, component, labels, owner, comments.
- Import from TestHub CSV, Zephyr Scale, Xray, TestRail or any CSV, or create draft tests from Jira issues with a JQL query. CSV export of everything, one folder (with subfolders) or unfiled tests.

**Configuration (open to everyone)**
- One page for environments, labels, iterations, priorities, shared datasets, custom statuses (tests, plans, cycles, executions) and custom fields (tests, plans, cycles, steps, executions). Custom statuses and priorities behave like a built-in one, so reports keep working. Built-in values can be archived; values in use cannot be deleted.

**Running tests**
- Cycles with progress, grouping, folders and paging. Quick Pass / Fail / Blocked, per-step results, actual results, comments, assignee, duration, evidence attachments and execution history.
- Create a Jira defect from a failed test, pre-filled with the failed step, expected and actual results.
- Re-run only the failed and blocked tests of a cycle.

**Reports**
- Dashboard, Summary, Cycle execution, Coverage, Traceability (issue to tests to results to defects), Trend, Burndown, and a Custom report builder with templates, saved reports and CSV export.

**Integrations**
- Jira (live issue status, search, defects, issue panel, TestHub page), Confluence (publish, link pages, macro), CI (JUnit and JSON results), test generators (create and read tests), webhooks (signed events), scheduled reports (Slack-compatible).

## How it is built (for technical readers)

```
 Browser ──────────────┐
 Jira (TestHub page) ──┼──▶ CloudFront ──▶ Application server ──▶ PostgreSQL
 CI / generators ──────┘       (HTTPS)      (Node.js, AWS ECS)    (Aurora)
        API key                                   │
                                                  ├──▶ Jira Cloud (REST API)
                                                  ├──▶ Confluence
                                                  └──▶ Webhooks / report URLs
```

- The Jira app is an Atlassian Forge app. Inside Jira it calls TestHub over HTTPS; Atlassian signs each call and TestHub verifies the signature, the app ID and the Jira site before answering.
- Each Jira user who opens TestHub inside Jira gets a TestHub account automatically (default role: tester).
- Data is stored in a PostgreSQL database. Secrets such as the Jira API token and webhook signing secrets are encrypted at rest.

## Roles

| Role | Can |
|---|---|
| Admin | Everything, including settings, users, API keys, webhooks and scheduled reports. |
| Tester | Write tests, run cycles and plans, record results, comment. |
| Viewer | Read only. |
| Approver (flag) | Approve or reject tests in review. Admins always can. |

## Known limits

- One Jira connection per TestHub (the site used for issue status, search and defects). The list of Jira projects allowed to use TestHub is global.
- No single sign-on or two-factor sign-in for the web app (inside Jira, people sign in through Jira).
- Scheduled reports go to a URL or Slack, not email.
- Webhook delivery is best effort: one retry, and a delivery can be lost if the server restarts.
- The database sleeps when idle, so the first request after a quiet period can take up to about 30 seconds.
