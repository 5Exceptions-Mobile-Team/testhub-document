# TestHub documentation

TestHub is 5Exceptions' test management platform: write test cases, run them in cycles, track results and report on quality. It works in a web browser and **inside Jira**, and other tools (CI pipelines, test generators) can connect to it through an API.

Platform address: https://d2i3n5h90346wi.cloudfront.net

## Read this first

| If you want to... | Read |
|---|---|
| Understand what TestHub is and what it can do | [1. Overview](docs/01-overview.md) |
| Learn to write tests, run cycles and read reports | [2. User guide](docs/02-user-guide.md) |
| Use TestHub inside your Jira, or connect a new Jira site | [3. Connect TestHub to Jira](docs/03-jira-integration.md) |
| Connect a tool, CI pipeline or test generator | [4. Connect apps (API)](docs/04-connect-apps.md) |
| Set up the platform, users, keys, webhooks and reports | [5. Administrator guide](docs/05-admin-guide.md) |
| Fix a problem | [6. Troubleshooting and FAQ](docs/06-troubleshooting.md) |

## Who does what

- **Testers and QA** use the web app or the TestHub page inside Jira (guide 2).
- **A Jira site admin** installs the TestHub app on their Jira site (guide 3, part B).
- **A TestHub admin** allows a new Jira site and project and creates API keys (guides 3 and 5).
- **Developers and tool owners** connect pipelines and generators with an API key (guide 4).

## Document status

Written 2026-11-04 for TestHub release `38dcb8e` and Jira app version 2.4.0. Where behaviour has a known limit, the guides say so.
