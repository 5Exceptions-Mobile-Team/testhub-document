# 4. Connect apps to TestHub (API)

This guide is for developers and tool owners: CI pipelines that report results, test generators that send test cases, and systems that want to react to what happens in TestHub.

- Base address: `https://d2i3n5h90346wi.cloudfront.net`
- Everything under `/api/ci/` uses an **API key**. Other `/api` routes belong to the web app and need a signed-in user; they are not for tools.

## 1. Get an API key

1. A TestHub **admin** opens **Settings → API keys** in the web app (or **Settings → Keys** inside Jira for the current project).
2. Choose the **project**, type a name (for example "GitHub Actions"), and click **Create key**.
3. Copy the token (it starts with `th_`). **It is shown once.** TestHub stores only a hash and cannot show it again.
4. Give it to the tool owner through a secret store (never in code or chat). Revoke it in the same screen at any time.

Rules for keys:
- A key belongs to **one project** and can only read or write that project. Using it on another project returns `403`.
- Keys created before project keys existed ("Legacy") work only for result uploads (`/api/ci/results` and `/api/ci/junit`). They are refused (`403`) everywhere else.
- Send the key as a header: `Authorization: Bearer th_...`
- Limit: **60 requests per minute per key**. Above that you get `429` and a `Retry-After` header.
- All requests must use HTTPS. Request bodies are limited to 2 MB for test cases.

## 2. Send automated test results

### JUnit XML

```bash
curl -X POST "https://d2i3n5h90346wi.cloudfront.net/api/ci/junit?project=TA&cycle=Nightly&createCycle=true&autoCreate=true&build=$BUILD_NUMBER" \
  -H "Authorization: Bearer $TESTHUB_TOKEN" \
  -H "Content-Type: application/xml" \
  --data-binary @target/surefire-reports/TEST-results.xml
```

### JSON

```bash
curl -X POST "https://d2i3n5h90346wi.cloudfront.net/api/ci/results" \
  -H "Authorization: Bearer $TESTHUB_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "project": "TA",
    "cycle": "Nightly",
    "createCycle": true,
    "autoCreate": true,
    "build": "1042",
    "environment": "staging",
    "results": [
      {"key": "TA-T1", "status": "Pass", "duration_ms": 1200},
      {"name": "Checkout fails on expired card", "status": "Fail", "comment": "HTTP 500"}
    ]
  }'
```

| Field | Meaning |
|---|---|
| `project` | Project key (required). |
| `cycle` | Cycle key (`TA-R3`) or name. With `createCycle: true` a missing cycle is created. Without a cycle, a new "Automated run" cycle is created. |
| `autoCreate` | Create library tests for results that match nothing (they get the label `automated` and Automation `Automated`). |
| `build`, `environment` | Stored on the cycle. |
| `results[].key` or `name` | How a result finds its test (see below). |
| `results[].status` | `Pass`, `Fail`, `Blocked`, `In Progress`; also `passed`, `failed`, `skipped` (becomes Blocked). |
| `results[].comment`, `duration_ms` | Optional. |

**Matching:** a test key such as `TA-T12` in the result key, name or class name wins; otherwise an exact (case-insensitive) name match. Unmatched results are listed in the response as `unmatched`.

**Response:** the cycle (`key`, `name`, `created`), counts of matched and created tests, and the cycle statistics. If the Jira integration is on and a test fails, TestHub can also create or reuse a Jira defect; pass `"jira": false` (JSON) or `&jira=false` (JUnit) to skip that.

## 3. Send generated test cases

For test generators (including AI tools). Cases are created in the **Library** as **Draft**, labelled `generated`, so a person reviews them before they count.

```bash
curl -X POST "https://d2i3n5h90346wi.cloudfront.net/api/ci/cases" \
  -H "Authorization: Bearer $TESTHUB_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "project": "TA",
    "folder": "Generated",
    "cases": [
      {
        "name": "Login works with a valid account",
        "objective": "A registered user can sign in",
        "priority": "High",
        "labels": ["login"],
        "automation": "Automated",
        "estimated_minutes": 5,
        "component": "Authentication",
        "steps": [
          {"step": "Open the login page", "expected": "The form is shown"},
          {"step": "Enter valid credentials and submit", "expected": "The dashboard opens"}
        ]
      }
    ]
  }'
```

- Up to **200 cases per call**, all created in one transaction: if one is invalid, none are saved and you get a `400` with the reason.
- `folder` defaults to `Generated` (created if missing). Names that already exist in the project (ignoring case and extra spaces) are **skipped**, not duplicated.
- The status is always **Draft**, whatever you send.
- Allowed `priority`: `Low`, `Normal`, `High`, `Critical`. Allowed `automation`: `None`, `Automated`, `Can't automate`, `Manual`. `script_type` is `steps` (default), `text` or `bdd`.
- Response (`201`): `{"created": ["TA-T101"], "skipped": [{"name": "...", "reason": "duplicate"}]}`.

### Read cases back

```bash
curl "https://d2i3n5h90346wi.cloudfront.net/api/ci/cases?project=TA&status=Draft&label=generated&limit=50" \
  -H "Authorization: Bearer $TESTHUB_TOKEN"
```

Returns `{"cases": [...], "next_cursor": "123"}`. Pass `cursor=123` for the next page. `limit` is 1 to 200 (default 50).

### Edit or delete a generated test

```bash
# change fields (only editable fields; status and folder cannot be changed)
curl -X PATCH "https://d2i3n5h90346wi.cloudfront.net/api/ci/cases/TA-T101?project=TA" \
  -H "Authorization: Bearer $TESTHUB_TOKEN" -H "Content-Type: application/json" \
  -d '{"priority": "Critical", "automation": "Automated"}'

# delete
curl -X DELETE "https://d2i3n5h90346wi.cloudfront.net/api/ci/cases/TA-T101?project=TA" \
  -H "Authorization: Bearer $TESTHUB_TOKEN"
```

Safety rule: a key may edit or delete **only tests that are still Draft and carry the `generated` label**. Approved tests and tests written by people are protected (`403 API keys can edit only generated Draft tests.`). Editable fields: `name, objective, precondition, priority, labels, automation, estimated_minutes, component, script_type, script_text, steps, custom`.

## 4. Read project structure and results

| Request | Returns |
|---|---|
| `GET /api/ci/cycles/{keyOrName}/summary?project=TA` | Cycle details, counts per status and every result. |
| `GET /api/ci/folders?project=TA` | Folders (`id`, `name`, `parent_id`, `case_count`). |
| `POST /api/ci/folders` with `{"project":"TA","name":"Payments","parent":"Generated"}` | Creates the folder (or returns the existing one with `200`). `parent` is an id or a name. |
| `GET /api/ci/plans?project=TA` | Plans with their progress. |
| `GET /api/ci/fields?project=TA` | Custom field definitions (so a tool can fill `custom`). |

## 5. Errors

| Code | Meaning |
|---|---|
| `400` | Invalid input. The message says what. Nothing was saved. |
| `401` | Missing, wrong or revoked key. |
| `403` | Key is for another project, a legacy key on a new endpoint, or the test is protected. |
| `404` | Project, cycle, folder or test not found. |
| `429` | More than 60 requests a minute. Wait for `Retry-After` seconds. |
| `5xx` | Server problem. The body contains a `ref` you can give to the TestHub owners. |

The first request after a quiet period can take up to about 30 seconds (the database wakes up), so use a timeout of at least 60 seconds in pipelines.

## 6. Webhooks: let TestHub call your system

TestHub can post a signed JSON message to your URL when something happens in a project. Admins manage them in the web app under **Settings → Webhooks & reports** (not inside Jira).

| Event | When |
|---|---|
| `execution.failed` | A test result changes to Fail. |
| `cycle.completed` | Every test in a cycle has a result. |
| `case.approved` | A test is approved. |
| `case.created` | A test is created (also for tests sent through the API, up to 50 per request). |

Requirements: the URL must be `https://` and public. Private, internal and cloud-metadata addresses, embedded credentials and redirects are refused. Up to 10 webhooks per project.

**Message**

```json
{
  "id": "5b0d4c0e-...",
  "event": "execution.failed",
  "created_at": "2026-11-04T10:15:00.000Z",
  "project": { "id": 3, "key": "TA", "name": "Truvideo" },
  "data": { "case_key": "TA-T12", "case_name": "...", "cycle_key": "TA-R3", "comment": "...", "executed_by": "..." }
}
```

**Headers:** `X-TestHub-Event`, `X-TestHub-Delivery` (a unique id; use it to ignore duplicates), `X-TestHub-Signature: t=<unix seconds>,v1=<hex>`.

**Verify the signature.** The signing secret (`whsec_...`) is shown once when the webhook is created. The signature is HMAC-SHA256 of `"<t>.<raw request body>"`.

```js
// Node.js
import crypto from 'node:crypto';
function verify(rawBody, header, secret) {
  const { t, v1 } = Object.fromEntries(header.split(',').map((p) => p.split('=')));
  const expected = crypto.createHmac('sha256', secret).update(`${t}.${rawBody}`).digest('hex');
  const fresh = Math.abs(Date.now() / 1000 - Number(t)) < 300;
  return fresh && crypto.timingSafeEqual(Buffer.from(expected), Buffer.from(v1));
}
```

```python
# Python
import hmac, hashlib, time
def verify(raw_body: bytes, header: str, secret: str) -> bool:
    parts = dict(p.split("=", 1) for p in header.split(","))
    expected = hmac.new(secret.encode(), f"{parts['t']}.".encode() + raw_body, hashlib.sha256).hexdigest()
    return abs(time.time() - int(parts["t"])) < 300 and hmac.compare_digest(expected, parts["v1"])
```

**Delivery:** answer with any `2xx` within 5 seconds. Otherwise TestHub retries once after 2 seconds. Delivery is best effort, so a message can be lost if the TestHub server restarts, and the same message can arrive twice. Make your receiver idempotent using `X-TestHub-Delivery`. The **Send test** button sends a `ping` event.

## 7. Scheduled reports

Under **Settings → Webhooks & reports → Scheduled reports** an admin can post a project summary (results of the last 7 days, failing tests, open cycles) to a URL daily or weekly at a chosen hour (UTC). A Slack incoming-webhook URL works as it is: the message has a `text` field. The same JSON also has a `report` object with the numbers for other tools. **Send now** tests it. There is no email delivery.

## 8. Putting it together: a typical pipeline

1. A TestHub admin creates a project key and gives the pipeline its token.
2. The pipeline runs tests and uploads JUnit or JSON results to a cycle named after the build (`createCycle=true`).
3. A generator posts new test ideas to `/api/ci/cases`; the QA team reviews the **Generated** tests in the Library and approves or archives them.
4. A webhook posts `execution.failed` to the team's chat tool; a weekly report goes to the QA channel.
5. In Jira, testers see the same data in the TestHub page and on each issue's Tests panel.
