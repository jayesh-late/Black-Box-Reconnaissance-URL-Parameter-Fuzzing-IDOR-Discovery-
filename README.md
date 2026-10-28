# Black-Box Reconnaissance & URL Parameter Fuzzing (IDOR Discovery)

This repository documents the Datheon Task 1 assessment of an API request that returned an application record when called with curl without explicit Cookie or Authorization headers.

Current conclusion
- Observed: GET /api/training/auth-app?email=... returned HTTP 200 and an application record in JSON.
- Authentication headers in the copied cURL request: No explicit Cookie or Authorization header was present.
- Preliminary finding: Potential unauthenticated data exposure.
- BOLA/IDOR status: Not independently confirmed. The captured test used the assessor's own record. No cross-user object access was tested.
- CVSS: A provisional 5.3 Medium estimate is included only under the assumption that an unauthenticated remote requester can read a limited amount of protected information. Reassess the metrics if further authorized evidence changes the impact.

This is a training assessment, not a claim that the issue has been confirmed across other users or records.

Repository contents
- report.md — disclosure-style assessment report.
- evidence/request-redacted.txt — sanitized reproduction command.
- evidence/response-redacted.json — response shape with personal and internal values redacted.
- evidence/observations.md — evidence log and limitations.

Reproduce the observed request
Use only the assigned training endpoint and your own assigned record. Substitute your own email locally; do not commit real personal data or credentials.

curl -i --compressed \
  'https://www.datheon.in/api/training/auth-app?email=YOUR_URL_ENCODED_EMAIL' \
  -H 'Accept: */*' \
  -H 'Referer: https://www.datheon.in/training/dashboard'
  
This command intentionally does not include Cookie or Authorization headers. A 200 response alone does not prove BOLA. Confirm the result is not dependent on another authentication mechanism, cache behavior, or intended public training functionality.

Safe handling
- Do not enumerate other students' email addresses or IDs.
- Do not upload unredacted screenshots, tokens, cookies, or private records.
- Do not claim cross-user IDOR unless access to an explicitly authorized test object belonging to another test account is demonstrated.
- Submit findings through the training platform's intended process.

References
- OWASP API1:2023 — Broken Object Level Authorization: https://api-security.owasp.org/editions/2023/en/0xa1-broken-object-level-authorization/
- OWASP Web Security Testing Guide — API BOLA testing: https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/12-API_Testing/02-API_Broken_Object_Level_Authorization/
- FIRST CVSS v3.1 Specification: https://www.first.org/cvss/v3-1/specification-document
