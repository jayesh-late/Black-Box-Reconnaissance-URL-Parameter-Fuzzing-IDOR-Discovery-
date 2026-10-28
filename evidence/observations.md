# Observation Log — Task 1: IDOR Discovery

## Assessment information

- **Target:** Datheon Student Workspace training application
- **Date observed:** 2026-10-28
- **Tools:** Firefox Developer Tools (Network panel), Kali Linux, cURL
- **Test type:** Read-only request inspection and replay for the assessor's own assigned record
- **Scope:** Assigned training dashboard and the assessor's own application record

## Observation 1 — Network request identified

- **HTTP method:** `GET`
- **Endpoint path:** `/api/training/auth-app`
- **Query parameter:** `email` (value intentionally redacted)
- **Observed status in browser:** `200`
- **Response content type:** JSON

The request was identified in Firefox Developer Tools under Network / Fetch-XHR while viewing the assigned dashboard.

**Suggested screenshot:** `screenshots/01-network-request.png`

## Observation 2 — cURL replay without explicit authentication headers

The request was copied as cURL and replayed from Kali Linux. The copied command did not contain an explicit `Cookie` header or an `Authorization: Bearer ...` header.

The replay returned:

- **Status:** `HTTP/2 200`
- **Content type:** `application/json`
- **Response structure:** `success: true` and an `applications` array
- **Data returned:** The assessor's own application record, including personal/application fields and internal-looking review/workflow metadata

The exact email, application identifier, personal details, and review-note values are intentionally omitted from this public log.

**Suggested screenshot:** `screenshots/03-curl-result-redacted.png`

## Observation 3 — Response headers

The captured response included the following headers:

- `content-type: application/json`
- `cache-control: public, max-age=0, must-revalidate`
- `x-vercel-cache: MISS`

These headers are recorded as observations only. They do not prove that a shared cache exposed private data. Cache behavior was not independently tested.

**Suggested screenshot:** If useful, include a separate redacted response-headers screenshot. Do not include cookies or credentials.

## Preliminary interpretation

The replay indicates potential unauthenticated data exposure because the endpoint returned the assessor's own application record without explicit cookie or bearer-token headers in the copied request.

This does **not yet confirm BOLA/IDOR**. The test accessed only the assessor's own record. No other user's record was requested, and no authorized second test account or cross-user test object was available in the evidence collected.

Other authentication mechanisms, intended training behavior, and endpoint access policy have not been independently ruled out.

## Evidence files

Store only sanitized evidence in the public repository:

- `request-redacted.txt` — safe reproduction template.
- `response-redacted.json` — redacted response structure.
- `screenshots/01-network-request.png` — browser request and status.
- `screenshots/02-api-response-redacted.png` — response structure with personal/internal values obscured.
- `screenshots/03-curl-result-redacted.png` — Kali cURL replay and status.
- `observations.md` — this observation log.

The screenshot files are not automatically generated. Add your own redacted screenshots using the filenames above.

## Limitations and safe handling

- Only the assessor's own assigned record was tested.
- No enumeration or cross-user access test was performed.
- No data was intentionally modified or deleted.
- No unredacted personal data, internal review notes, cookies, bearer tokens, or session secrets should be committed.
- Do not claim that other users' records are exposed unless an explicitly authorized test demonstrates that result.

## Current status

**Potential unauthenticated data exposure observed; BOLA/IDOR not independently confirmed.**
