# 2. User guide

This guide is for testers, QA and test leads. It applies to the web app and to the TestHub page inside Jira; where they differ it says so.

- Web app: https://d2i3n5h90346wi.cloudfront.net
- In Jira: open a project and choose **TestHub** in the left sidebar. You are signed in through Jira. (If you do not see it, see [guide 3](03-jira-integration.md).)

The top bar has **Dashboard, Test Library, Test Cycles, Test Plans, Reports, Settings** and a project picker. Pick your project first.

## 1. Getting started in five steps

1. An admin creates your project (name and key) and sets its **Jira project key** (Settings → Project).
2. Admins add people (Settings → Users), or people are added automatically the first time they open TestHub inside Jira.
3. Write test cases in the **Test Library**, or import them (section 8).
4. Create a **cycle**, add tests to it, and run it (section 4).
5. Watch progress on the **Dashboard** and in **Reports**.

## 2. Test Library

**Folders (left panel)**
- Click the arrow to expand or collapse a folder. The tree remembers what you opened.
- The **⋯** menu on a folder: Add subfolder, Rename, Delete, Expand all, Collapse all, **Create test cycle** (from every test in the folder and its subfolders), the folder **ID** (click to copy) and **Copy folder link** (web app).
- The 🔍 button above the tree finds folders by name. The **＋** button creates a top-level folder.
- Drag test cases from the list onto a folder to move them.
- **All test cases** shows everything; **Unfiled** shows tests without a folder.

**The list**
- Search matches name, key, label, Jira issue and the text of steps, objective and precondition.
- Filters: status, priority, automation, component, and a **Generated** button that shows tests sent by external tools.
- **Columns** shows or hides: Priority, Status, Automation, Labels, Owner, Component, Est. min, Steps, Last result, Jira, Updated, and your custom fields.
- Click a column heading to sort. Choose how many rows per page at the bottom.
- If the table is wider than the screen, it scrolls on its own; the top bar and folder panel stay in place.
- **Export CSV** exports the current selection: the selected folder with its subfolders, only Unfiled, or everything.
- Tick rows to use bulk actions: **Add to cycle**, **Clone**, **Archive**, **Move to…** (a folder), **Priority…**, and **Delete** (admins only).

**Create a test case**
1. Click **New test case**.
2. Fill in **Name** (required), Folder, Priority, Owner, **Automation**, Component, Estimated time (minutes), Objective, Precondition and Labels.
3. Link Jira issues (the requirements or stories the test covers). **Search Jira** finds issues by key or words in the title.
4. Add steps. Each step has the action, optional test data and the expected result. You can reorder or remove steps. You can also write the test as plain text or as BDD (Given/When/Then).
5. Click **Create**.

**Open an existing test** to see these tabs: Details, **Versions**, **Executions**, **Comments** and (when the review workflow is on) Review history. The footer has **Delete**, **Clone**, **Archive**, Close and Save changes.

| Automation value | Meaning |
|---|---|
| None | Not decided yet (the default). |
| Automated | An automated test exists. |
| Can't automate | It cannot reasonably be automated. |
| Manual | It is run by hand. |

**Reusable steps and data**
- A step can **call another test**. Its steps are included live when the test runs, and the called test shows "used by".
- Put `{{name}}` in a step, add a **dataset** (a table of values, with CSV import and export), and each row runs as its own iteration with its own result.

**Versions.** Changing the name, objective, precondition or steps creates a new version. Open the Versions tab to compare and **restore** an older one. Changing only the automation status, estimate, component, labels, owner or priority does not create a new version. An execution remembers which version it ran.

**Review workflow** (when an admin turns it on for the project)
1. The author submits a Draft for review.
2. An approver approves it or rejects it with a reason. Every step is logged.
3. Editing an approved test can send it back to Draft, and tests can be limited to Approved when added to cycles, depending on the project settings.

**Comments.** Use the Comments tab to discuss a test. You can delete your own comments; admins can delete any.

## 3. Archive and delete

- **Archive** sets a test to Deprecated and leaves it out of new cycles. Use it from the test editor, or in bulk from the list. Filter by status Deprecated to see archived tests.
- **Delete** removes a test and its results permanently. Only admins can delete in bulk.

## 4. Test Cycles: running tests

**Create a cycle.** Click **New cycle**. Fill in Name, Build / version, Environment, Start and End dates, Test plan and Folder. Open the cycle and use **Add cases** to choose tests by folder or by search. You can also create a cycle straight from a Library folder (folder ⋯ menu → Create test cycle) or from selected tests (**Add to cycle**).

**The cycle list.** Search, sort, group by status, plan, build or environment, organise cycles in folders (drag a cycle onto a folder), and page through the results. The play button opens the cycle ready to run.

**Run tests**
- Quick results: **Pass**, **Fail** or **Blocked** on a row.
- Open a test (**Execute**) for the full panel: overall status, a result and actual result for each step, comment, assignee, duration, evidence attachments (screenshots, logs) and history. **Save & next** moves on to the next test; **Previous** and **Next** move around.
- A test with a dataset shows one block per data row. **Pass all steps** marks every step as passed.
- Select several rows to **assign**, change the status of many tests at once, or remove tests from the cycle.
- **Export CSV** downloads the cycle results.

**Failed tests**
- **Create defect** (in the execution panel) raises a Jira issue pre-filled with the failed step, the expected and actual results and your comment. You choose the issue type (Auto prefers Bug). It is linked to the stories the test covers. You can also paste an existing defect key.
- **Re-run failed** (on a cycle that has failures) creates "<name> (re-run)" containing only the failed and blocked tests, with the same plan, folder, build and environment.

When every test in a cycle has a result the cycle becomes Done automatically.

## 5. Test Plans

A plan groups cycles, for example one release. Open **Test Plans → New plan**, add existing cycles or create new ones. The plan shows the combined progress of its cycles.

## 6. Dashboard

Counts and progress for the project, results, tests failing now, tests waiting for review, tests assigned to you, recent activity, the trend, open cycles and the burndown. Click an item to open it.

## 7. Reports

| Tab | What it shows |
|---|---|
| Summary | Tests by status and priority, recent cycles with progress. |
| Cycle execution | One cycle by result and priority, with the failed and blocked list. |
| Coverage | Which tests are linked to Jira issues and which have been run. |
| Traceability | Jira issue to tests to latest result to defects. |
| Trend | Results per day over a chosen period. |
| Burndown | Remaining tests per day against an ideal line, with scope changes and a projected finish, for a cycle, a plan or all active cycles. |
| Custom | Build your own: choose the source (executions or tests), group by any field (including custom fields), add filters and a date range. Save the report, save a copy, or export CSV. Templates give a quick start. |

## 8. Importing tests

Open the import wizard from the Test Library.

- **TestHub CSV**, or a CSV from **Zephyr Scale**, **Xray** or **TestRail**, or any CSV with a header row. Map the columns, choose the mode, preview, then import. An Excel file must be saved as CSV first.
- **From Jira issues:** enter a JQL query and TestHub creates draft tests from the matching issues.

## 9. Inside Jira

- **TestHub page:** the whole application in the project sidebar. Use it exactly like the web app.
- **Tests panel:** on a Jira issue, shows the tests covering it and their latest results, and (if enabled by an admin) a button to create a draft test from the issue.
- Publishing to Confluence and Confluence searches are available in the web app only.
- Server-wide settings (Jira connection, Jira app, Confluence, webhooks, scheduled reports) are in the web app. Inside Jira, admins can use Settings → Project, Custom fields, Users and Keys for the current project.

## 10. Tips

- Use labels and components to slice the library; use folders for structure.
- Use datasets instead of copying a test many times.
- Use **Generated** to review tests that came from tools, then approve or archive them.
- Link tests to Jira issues so Coverage and Traceability show something useful.
- The very first page load after a quiet period can take up to about 30 seconds, because the database sleeps when idle.
