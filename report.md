# Responsible Vulnerability Disclosure Report

## 1. Assessment Summary

| Field | Details |
|---|---|
| Assessment | Black-Box Reconnaissance & URL Parameter Fuzzing (IDOR Discovery) |
| Target | Datheon Student Workspace — assigned training environment |
| Observed endpoint | `GET /api/training/auth-app?email=[REDACTED]` |
| Assessment date | 2026-10-28 |
| Testing environment | Firefox Developer Tools and Kali Linux |
| Preliminary classification | Potential unauthenticated data exposure |
| BOLA/IDOR status | Not independently confirmed |
| Severity | Provisional only; see Section 8 |

## 2. Executive Summary

As part of the assigned Datheon training exercise, the dashboard's network traffic was inspected in Firefox Developer Tools. A GET request to `/api/training/auth-app` was identified with an `email` query parameter.

The request was copied as cURL and replayed from Kali Linux without explicit `Cookie` or `Authorization` headers. The server returned `HTTP/2 200` and a JSON response containing the assessor's own application record. The response included personal/application fields and internal-looking review/workflow metadata.

This observation raises a potential unauthenticated data exposure concern if the endpoint is intended to return this information only to authenticated or otherwise authorized users. The evidence currently available does **not** establish cross-user Broken Object Level Authorization (BOLA/IDOR), because only the assessor's own record was requested and observed.

## 3. Scope and Authorization

### In scope

- Inspecting the assigned Datheon training dashboard.
- Reviewing the dashboard's browser-generated API request.
- Replaying the request for the assessor's own assigned record without explicitly adding cookie or bearer-token headers.
- Capturing and documenting sanitized evidence.

### Out of scope / not performed

- Enumerating other students' email addresses or application identifiers.
- Attempting to access another user's private record.
- Sending state-changing requests.
- Testing administrative functions.
- Confirming whether a shared cache exposes personalized responses.
- Establishing whether the endpoint's behavior is intended by the training application.

Testing was limited to the assigned training environment and the assessor's own record.

## 4. Technical Details

### Observed request

```http
GET /api/training/auth-app?email=[REDACTED]
Host: www.datheon.in
Accept: */*
Referer: https://www.datheon.in/training/dashboard
```

The captured cURL request did not contain explicit `Cookie` or `Authorization` headers.

### Observed response

- **Status:** `HTTP/2 200`
- **Content type:** `application/json`
- **Server:** `Vercel`
- **Response body:** JSON with `success: true` and an `applications` array containing the assessor's own application record.
- **Observed response fields:** Application identifier/number, name, email, application status, education details, profile links, and review/workflow metadata. Values are redacted in the public evidence.

The response also included `cache-control: public, max-age=0, must-revalidate` and `x-vercel-cache: MISS`. These headers should be reviewed, but they do not by themselves prove that private information was served from a shared cache.

## 5. Steps to Reproduce

Perform this reproduction only for the assessor's own assigned record.

1. Sign in to the assigned Datheon training workspace and open the dashboard.
2. Open Firefox Developer Tools, select **Network**, and filter to **Fetch/XHR**.
3. Select the request to `/api/training/auth-app` and inspect its headers and response.
4. Choose **Copy → Copy as cURL**.
5. In Kali Linux, replay a sanitized version without adding cookie or authorization headers:

```bash
curl -i --compressed \
  'https://www.datheon.in/api/training/auth-app?email=YOUR_URL_ENCODED_EMAIL' \
  -H 'Accept: */*' \
  -H 'Referer: https://www.datheon.in/training/dashboard'
```

6. Record the HTTP status and whether the response includes the assigned record. Do not publish an unredacted response or real personal information.

### Actual observed result

The replay returned `HTTP/2 200` and JSON containing the assessor's own application record. No explicit cookie or bearer-token header was present in the copied cURL request.

## 6. Evidence

Place the following sanitized screenshots in `evidence/screenshots/` before publishing the repository:

| File | What it should demonstrate |
|---|---|
| `01-network-request.png` | Firefox Network panel showing the request and HTTP status |
| `02-api-response-redacted.png` | Response structure with personal and internal fields redacted |
| `03-curl-result-redacted.png` | Kali terminal showing the sanitized cURL test and status code |

Supporting text evidence:

- `evidence/request-redacted.txt` — sanitized cURL reproduction template.
- `evidence/response-redacted.json` — redacted representation of the response structure.
- `evidence/observations.md` — evidence log and test limitations.

Only add screenshots after creating the named files. Keep originals privately and publish redacted copies. Do not expose cookies, access tokens, personal data, or internal review notes.

## 7. Security Impact and Classification

### Potential impact

If the endpoint returns non-public application records to unauthenticated remote requesters, a requester who knows or obtains a valid email parameter may be able to retrieve information associated with that record. Depending on the intended access policy and sensitivity of the fields, this could expose personal information or internal application/workflow metadata.

### Current classification

- **Preliminary concern:** Potential unauthenticated data exposure.
- **Related OWASP category:** API1:2023 Broken Object Level Authorization (BOLA), if unauthorized access to specific protected objects is demonstrated.
- **BOLA/IDOR confirmation:** Not established by the current evidence.

A request returning the assessor's own record does not, on its own, prove IDOR. Confirmation requires evidence that the server fails to enforce object-level permissions for an object the requester is not authorized to access. The task's training instructions should be followed for any further authorized validation.

## 8. CVSS v3.1 Severity Estimate

**Provisional score: 5.3 (Medium)**  
**Provisional vector:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N`

This estimate is conditional on the following assumptions: the endpoint is network-accessible; exploitation requires no special conditions, privileges, or victim interaction; the demonstrated impact is limited confidentiality loss; and no integrity or availability impact has been demonstrated.

This is **not a final severity determination**. If the endpoint is intended to be public, if an additional authentication mechanism applies, or if the actual confidentiality impact differs from the assumptions, the score must be reassessed. Do not present the provisional score as confirmed severity without validating the endpoint's intended access policy and impact.

Use the official [FIRST CVSS v3.1 Calculator](https://www.first.org/cvss/calculator/3.1) to verify or revise the vector.

## 9. Remediation Recommendations

1. Require appropriate authentication for endpoints that return non-public application records.
2. Enforce server-side authorization for each requested object using the authenticated principal and the specific object.
3. Do not treat a client-supplied email address or identifier as proof of permission.
4. Return only the fields required by the dashboard; exclude internal review notes, workflow identifiers, and other nonessential metadata from client-facing responses.
5. Review caching behavior for personalized or sensitive API responses and apply an appropriate private/no-store policy where required by the application's design.
6. Add automated tests for unauthenticated requests, cross-account access denial, and response-field minimization.
7. Log authorization failures without recording credentials or sensitive response bodies.

## 10. Conclusion

**Status: Potential issue observed; BOLA/IDOR not yet confirmed.**

A cURL replay without explicit cookie or bearer-token headers returned the assessor's own application record. Further authorized validation, or confirmation from the training application's owner, is needed to determine whether this behavior violates the application's intended access-control policy and whether other protected objects are accessible without authorization.

## 11. References

- OWASP API1:2023 — Broken Object Level Authorization: https://api-security.owasp.org/editions/2023/en/0xa1-broken-object-level-authorization/
- OWASP Web Security Testing Guide — API BOLA testing: https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/12-API_Testing/02-API_Broken_Object_Level_Authorization/
- FIRST CVSS v3.1 Specification: https://www.first.org/cvss/v3-1/specification-document
- FIRST CVSS v3.1 Calculator: https://www.first.org/cvss/calculator/3.1
