**API Security Risk Analysis Report**  
**Future Interns — Cyber Security Track | Task 3**  
   
 **Repository:** FUTURE_CS_03  
| | |  
|-|-|  
| **Field** | **Detail** |   
| Target API | [e.g. https://gorest.co.in/public/v2]*(public test API)* |   
| Assessment type | Black-box API review, non-destructive |   
| Standard applied | OWASP API Security Top 10 (2023) |   
| Assessed by | [Shivansh Yadav] — Cyber Security Intern, Future Interns |   
| CIN ID | [FIT/SEP26/CS10303] |   
| Date | [27/09/26] |   
| Version | 1.0 |   
   
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANUlEQVR4nO3OMQ2AABAAsSPBCUZfEnoYmFDBhAU2QtIq6DIzW7UHAMBfnGt1V8fXEwAAXrse/wcF74lXkIsAAAAASUVORK5CYII=)  
**⚠️ Scope and Authorisation**  
Test only APIs that explicitly permit it. Safe choices:  
| | | |  
|-|-|-|  
| **API** | **Why it's usable** | **Base URL** |   
| **GoRest** | Public demo API with real auth tokens and full CRUD | gorest.co.in/public/v2 |   
| **Reqres** | Hosted test API for request/response practice | reqres.in/api |   
| **JSONPlaceholder** | Read-mostly fake REST API | jsonplaceholder.typicode.com |   
| **DVWS / vAPI / crAPI (self-hosted)** | Deliberately vulnerable APIs built for training — the best option | OWASP/GitHub |   
| **Your own API** | You own it | — |   
   
***crAPI*** * (Comple* *tely Ridiculous API, from OWASP) is the strongest choice for this task: it is designed so that every OWASP API Top 10 category is genuinely present and findable, which means your report contains real verified findings rather than placeholders. Run it with * *docker compose* * locally.*  
**Out of scope:** load or denial-of-service testing, exploiting other users' real data, brute-forcing credentials, and any destructive operation on a shared public API.  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANUlEQVR4nO3OQQmAABRAsSd4NIGRTPXNaQBrWMGbCFuCLTOzV2cAAPzFvVZbdXw9AQDgtesBhZQEOYZGgUEAAAAASUVORK5CYII=)  
**1. Executive Summary**  
APIs now carry the majority of application traffic, and they fail differently from web pages. A website hides what a user should not see; an API often returns everything and trusts the client to hide it. That gap is where most API breaches come from.  
This assessment reviewed [N] endpoints of [target API] against the **OWASP API Security Top 10 (2023)**.  
**[N]** ** issues were identified: ** **[x]** ** High, ** **[y]** ** Medium, ** **[z]** ** Low.**  
The most significant finding was [one-line description], which would allow [business consequence].  
**Recurring theme:** [e.g. "the API authenticates users correctly but does not check whether the authenticated user is allowed to access the specific record requested"]. Authentication asks *who are you*; authorisation asks  *are you allowed this particular thing*. Most of the findings below are failures of the second question.  
**Findings at a glance**  
| | | | |  
|-|-|-|-|  
| **ID** | **Finding** | **OWASP API** | **Severity** |   
| API-001 | Broken Object Level Authorization (BOLA/IDOR) | API1:2023 | 🔴 High |   
| API-002 | Excessive data exposure in responses | API3:2023 | 🔴 High |   
| API-003 | No rate limiting / unrestricted resource consumption | API4:2023 | 🟠 Medium |   
| API-004 | Weak authentication and token handling | API2:2023 | 🔴 High |   
| API-005 | Missing input validation / mass assignment | API3, API6 | 🟠 Medium |   
| API-006 | Security misconfiguration (CORS, headers, verbose errors) | API8:2023 | 🟠 Medium |   
| API-007 | Unauthenticated or undocumented endpoints exposed | API9:2023 | 🟡 Low |   
   
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANElEQVR4nO3OQQmAABRAsad4EjtY9fewnUms4E2ELcGWmTmrKwAA/uLeqrU6vp4AAPDa/gDzWAM6QQXRdAAAAABJRU5ErkJggg==)  
**2. Methodology**  
**2.1 Phases**  
| | | |  
|-|-|-|  
| **Phase** | **Activity** | **Tooling** |   
| 1. Discovery | Map endpoints, methods, parameters | Documentation, Swagger/OpenAPI spec, browser DevTools Network tab, JS bundle review |   
| 2. Baseline | Capture normal authenticated behaviour | Postman collection |   
| 3. Authentication testing | Test token issuance, expiry, and validation | Postman, jwt.io |   
| 4. Authorisation testing | Attempt cross-user and vertical access | Two test accounts in Postman |   
| 5. Data exposure review | Compare what is returned vs. what the UI shows | Response diffing |   
| 6. Input & rate testing | Malformed input, boundary values, request bursts | Postman Runner |   
| 7. Configuration review | Headers, CORS, TLS, error verbosity | curl, DevTools |   
| 8. Reporting | Document and prioritise | This document |   
   
**2.2 Environment setup**  
**Create two test accounts.** Almost every serious API finding requires comparing what User A can do to User B's data. A single account cannot demonstrate BOLA.  
User A: id = 101, token = {{tokenA}}  
 User B: id = 202, token = {{tokenB}}  
   
**Postman configuration**  
1. Create an environment with variables: baseUrl, tokenA, tokenB, userAId, userBId.  
2. Set collection-level auth to Bearer {{tokenA}} so it inherits everywhere.  
3. Enable the **Postman Console** (View → Show Postman Console) to see raw requests and responses.  
4. Save every request into a collection and **export it to the repo** — the collection is part of your evidence.  
**Equivalent curl baseline**  
export BASE="https://gorest.co.in/public/v2"  
 export TOKEN_A="..."  
 export TOKEN_B="..."  
   
 # Baseline authenticated request  
 curl -s -H "Authorization: Bearer $TOKEN_A" "$BASE/users/101" | jq  
   
 # Same request, no token  
 curl -s -o /dev/null -w "%{http_code}\n" "$BASE/users/101"  
   
 # User A's token requesting User B's record  ← the BOLA test  
 curl -s -H "Authorization: Bearer $TOKEN_A" "$BASE/users/202" | jq  
   
**2.3 Finding hidden endpoints**  
Undocumented endpoints are frequently the least protected ones.  
- **JS bundles:** open DevTools → Sources, search the bundled JavaScript for /api/, fetch(, axios..  
- **OpenAPI spec:** try /swagger.json, /openapi.json, /v2/api-docs, /swagger-ui.html, /redoc.  
- **Mobile app traffic:** proxy through Burp Suite to see calls the web UI never makes.  
- **Version drift:** if /v2/users exists, check /v1/users — old versions often outlive their security patches.  
- **Method probing:** an endpoint that allows GET may also silently allow PUT or DELETE.  
curl -s -X OPTIONS -i "$BASE/users/101" | grep -i allow  
   
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANklEQVR4nO3OQQmAABRAsScYxpg/h5VMYARvRrCCNxG2BFtmZquOAAD4i3Ot7mr/egIAwGvXA224BcUMk6pDAAAAAElFTkSuQmCC)  
**3. Risk Classification**  
| | | |  
|-|-|-|  
| **Severity** | **Definition** | **Fix window** |   
| 🔴 **High** | Direct unauthorised access to, or modification of, data belonging to other users; authentication bypass | 7 days |   
| 🟠 **Medium** | Significantly increases attack feasibility or exposes internal detail; needs another condition to be fully exploited | 30 days |   
| 🟡 **Low** | Hygiene and hardening issue with limited standalone impact | Next release |   
   
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANklEQVR4nO3OMQ2AABAAsSNBCkLfFDZwwIgHRiywEZJWQZeZ2ao9AAD+4lyruzq+ngAA8Nr1AOH0BedHjjlfAAAAAElFTkSuQmCC)  
**4. Detailed Findings**  
*Templates below cover the vulnerability classes you are most likely to confirm. * ***Keep only what you actually verified*** *, replace the evidence blocks with your real Postman screenshots and responses, and delete the rest.*  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANElEQVR4nO3OMQ0AIAwAwZIgBKn1gjJsdGLBABMhuZt+/JaZIyJmAADwi9VP1NMNAABu1AaU4gUeBSGW2wAAAABJRU5ErkJggg==)  
**API-001 — Broken Object Level Authorization (BOLA / IDOR)**  
| | |  
|-|-|  
|   |   |   
| **Severity** | 🔴 **High** |   
| **OWASP API** | API1:2023 – Broken Object Level Authorization |   
| **CWE** | CWE-639 |   
| **Endpoint** | GET /users/{id}, PUT /users/{id}, DELETE /users/{id} |   
   
**What it is (plain language)**  
   
 The API checks that you are logged in, but not that the record you asked for is yours. Change the number in the URL and you get someone else's data.  
**Why it matters to the business**  
   
 This is the single most common cause of real API breaches, because it requires no skill to exploit — an attacker changes one digit. Since record IDs are usually sequential, the entire user base can be downloaded by a script in minutes. In practical terms: every customer's personal data, in one file, taken by anyone with a free account.  
**Evidence**  
GET /public/v2/users/202  
 Authorization: Bearer {{tokenA}}       ← User A's token  
   
 HTTP/1.1 200 OK  
 {  
   "id": 202,  
   "name": "User B",  
   "email": "userb@example.com",  
   "phone": "+91XXXXXXXXXX"  
 }  
   
*Expected: * *403 Forbidden* * or * *404 Not Found* *.*  
 *  
 Screenshot: * *evidence/api-001-bola.png*  
**Remediation**  
1. **Enforce an ownership check on every object access**, server-side, in the data layer:  
2. # Vulnerable  
 user = User.objects.get(id=requested_id)  
   
 # Fixed — the query itself cannot return another user's record  
 user = User.objects.get(id=requested_id, owner=request.user)  
   
3. Prefer **deriving the ID from the session** where possible: GET /me/profile instead of GET /users/{id}.  
4. Replace sequential integer IDs with **UUIDs**. This is obfuscation, not a fix — do it *in addition to* the authorisation check, never instead of it.  
5. Centralise authorisation in middleware or a policy layer so it cannot be forgotten on a new endpoint.  
6. Add an automated test per endpoint: *User A must receive 403 for User B's object.* Run it in CI.  
**Verification** Repeat the request with User A's token against User B's ID; a 403 or 404 confirms the fix. Test GET, PUT, PATCH, and DELETE separately — read may be fixed while write is not.  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANUlEQVR4nO3OMQ2AABAAsSNhYMMAKlD4OzrxgQU2QtIq6DIzR3UFAMBf3Gu1VefXEwAAXtsfSqADWz4G/HUAAAAASUVORK5CYII=)  
**API-002 — Excessive data exposure**  
| | |  
|-|-|  
|   |   |   
| **Severity** | 🔴 **High** |   
| **OWASP API** | API3:2023 – Broken Object Property Level Authorization |   
| **CWE** | CWE-213 |   
| **Endpoint** | GET /users, GET /users/{id} |   
   
**What it is**  
   
 The API returns the complete database record and relies on the app's screen to display only some of it. Anyone reading the raw response sees everything.  
**Why it matters**  
   
 The website or mobile app is not a security boundary — it is just one client. Open DevTools and the hidden fields are right there. Password hashes, internal flags, other users' email addresses, and admin markers routinely leak this way.  
**Evidence**  
GET /users/101  
 {  
   "id": 101,  
   "name": "Test User",  
   "email": "test@example.com",  
   "password_hash": "$2b$12$...",      ← must never leave the server  
   "role": "user",  
   "internal_notes": "flagged for review",  
   "is_admin": false,  
   "api_key": "sk_live_..."            ← credential disclosure  
 }  
   
**Remediation**  
1. **Define an explicit response schema (DTO/serializer) per endpoint.** Allow-list the fields to return; never serialise the model object directly.  
2. // Vulnerable  
 res.json(user);  
   
 // Fixed  
 res.json({ id: user.id, name: user.name, email: user.email });  
   
3. Never include credentials, hashes, tokens, or internal flags in any response.  
4. Apply **field-level authorisation** — an admin may see internal_notes; a user may not. Same endpoint, different schema.  
5. Add a CI test asserting that responses contain no key matching /password|secret|token|key|hash/i.  
6. Review responses of *every* endpoint, including error responses and list endpoints (which often leak more than detail endpoints).  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANUlEQVR4nO3OQQ2AQBAAsSHhiQI0IWp9ngBsYIEfIWkVdJuZs5oAAPiLe6+O6vp6AgDAa+sBhYwEOqBD7p8AAAAASUVORK5CYII=)  
**API-003 — Unrestricted resource consumption (no rate limiting)**  
| | |  
|-|-|  
|   |   |   
| **Severity** | 🟠 **Medium** (🔴 High if it fronts login, OTP, or a paid service) |   
| **OWASP API** | API4:2023 – Unrestricted Resource Consumption |   
| **CWE** | CWE-770 |   
| **Endpoint** | All, notably POST /login and POST /otp/verify |   
   
**What it is**  
   
 The API accepts an unlimited number of requests from the same client.  
**Why it matters**  
   
 Without a limit, a 6-digit OTP has only one million possibilities — a script exhausts them in minutes and account security collapses regardless of how strong passwords are. Beyond authentication: unlimited requests mean unlimited data scraping, unlimited SMS/email cost on your account, and a cheap route to taking the service down.  
**Evidence**  
for i in $(seq 1 200); do  
   curl -s -o /dev/null -w "%{http_code} " \  
     -H "Authorization: Bearer $TOKEN_A" "$BASE/users"  
 done  
 # Output: 200 200 200 ... (200 times) — no 429 returned  
   
*Use Postman Runner with 200 iterations as the GUI equivalent.*  
 *  
 * ***Note:*** * on a shared public API, keep volume modest. The point is to demonstrate absence of a limit, not to stress the service.*  
**Remediation**  
1. Apply **per-user, per-IP, and per-endpoint rate limits**. Authentication endpoints need far tighter limits than read endpoints.  
2. Return 429 Too Many Requests with a Retry-After header and rate-limit headers (X-RateLimit-Remaining).  
3. Add **progressive delays and account lockout** on repeated authentication failures.  
4. Enforce **pagination with a maximum page size** — reject ?limit=1000000.  
5. Cap request body size, upload size, JSON nesting depth, and array lengths.  
6. For GraphQL, add query depth and complexity limits.  
7. Set execution timeouts so one expensive query cannot hold resources indefinitely.  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANUlEQVR4nO3OQQmAABRAsSd49m4tA8nPaQJjWMGbCFuCLTOzV2cAAPzFvVZbdXw9AQDgtesBorcEPwOKyvQAAAAASUVORK5CYII=)  
**API-004 — Weak authentication and token handling**  
| | |  
|-|-|  
|   |   |   
| **Severity** | 🔴 **High** |   
| **OWASP API** | API2:2023 – Broken Authentication |   
| **CWE** | CWE-287, CWE-798 |   
   
**What it is**  
   
 Problems in how the API issues, validates, and retires its access tokens.  
**Checks performed**  
| | | |  
|-|-|-|  
| **Check** | **Method** | **Result** |   
| Request with no token | Remove Authorization header | [ ] |   
| Request with malformed token | Bearer invalid123 | [ ] |   
| Token in URL query string | Inspect request format | [ ] |   
| JWT alg: none accepted | Re-sign with none, resend | [ ] |   
| JWT signature not verified | Alter payload, keep old signature | [ ] |   
| Token expiry enforced | Check exp claim; wait and retry | [ ] |   
| Token revoked on logout | Log out, reuse old token | [ ] |   
| API key hard-coded in client | Search JS bundle for key/secret | [ ] |   
| Credentials over HTTP | Check scheme on auth endpoints | [ ] |   
   
**Inspecting a JWT**  
# Decode header and payload (they are only base64, not encrypted)  
 echo "$JWT" | cut -d. -f1 | base64 -d 2>/dev/null | jq  
 echo "$JWT" | cut -d. -f2 | base64 -d 2>/dev/null | jq  
   
Look for: "alg": "none", "alg": "HS256" where RS256 was expected (key-confusion risk), a missing exp claim, and sensitive data in the payload — **a JWT payload is readable by anyone holding the token**.  
**Why it matters**  
   
 A token is a bearer credential: whoever holds it *is* the user. If tokens never expire, a single leak from a log file, a browser history entry, or a shared screenshot grants permanent access.  
**Remediation**  
1. Use **short-lived access tokens** (5–15 minutes) with rotating refresh tokens.  
2. **Always verify the signature** and explicitly pin the expected algorithm — never trust the alg header from the token itself.  
3. Maintain a **revocation list** so logout genuinely invalidates a token.  
4. Never place tokens in URLs — they end up in server logs, proxy logs, and Referer headers. Use the Authorization header.  
5. Never ship secrets in client-side code or mobile apps; anything shipped to a client is public.  
6. Enforce HTTPS everywhere with HSTS.  
7. Add MFA for sensitive operations and rate-limit all authentication endpoints (see API-003).  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANklEQVR4nO3OQQmAABRAsSfYxZo/jkUsYQLPJrCCNxG2BFtmZquOAAD4i3Ot7mr/egIAwGvXA4rDBc72meO5AAAAAElFTkSuQmCC)  
**API-005 — Missing input validation and mass assignment**  
| | |  
|-|-|  
|   |   |   
| **Severity** | 🟠 **Medium** (🔴 High if privilege fields are writable) |   
| **OWASP API** | API3:2023, API6:2023 |   
| **CWE** | CWE-915, CWE-20 |   
   
**What it is**  
   
 The API takes the JSON it receives and writes all of it to the database, including fields the user was never supposed to control.  
**Why it matters**  
   
 If is_admin or role is writable, a normal user can promote themselves to administrator by adding one line to their profile-update request. There is no exploit code involved — it is a valid, well-formed API call.  
**Evidence**  
PATCH /users/101  
 Authorization: Bearer {{tokenA}}  
 Content-Type: application/json  
   
 { "name": "Test User", "role": "admin", "is_admin": true, "balance": 999999 }  
   
 HTTP/1.1 200 OK  
 { "id": 101, "name": "Test User", "role": "admin", "is_admin": true }  
   
*Extra fields were accepted and persisted.*  
**Additional input validation tests**  
| | | |  
|-|-|-|  
| **Input** | **Expected** | **Actual** |   
| "age": -5 | 400 rejected | [ ] |   
| "age": "twenty" | 400 type error | [ ] |   
| "email": "notanemail" | 400 format error | [ ] |   
| 10,000-character name | 400 length error | [ ] |   
| "name": "<script>alert(1)</script>" | Stored encoded / rejected | [ ] |   
| "id": 999 (path says 101) | Path value wins | [ ] |   
| Deeply nested JSON (100 levels) | 400 rejected | [ ] |   
   
**Remediation**  
1. **Allow-list writable fields explicitly** — never bind the whole request body to the model.  
2. // Vulnerable  
 await User.update(req.body, { where: { id } });  
   
 // Fixed  
 const { name, phone } = req.body;  
 await User.update({ name, phone }, { where: { id } });  
   
3. Validate with a **schema** (JSON Schema, Zod, Joi, Pydantic) and reject unknown properties outright.  
4. Enforce types, ranges, lengths, and formats server-side. Client-side validation is a usability feature, not a control.  
5. Keep privilege fields (role, is_admin, balance, verified) in a separate, admin-only update path.  
6. Use parameterised queries throughout — API endpoints reach the same databases as web forms and are just as injectable.  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANUlEQVR4nO3OQQmAABRAsSeYxKS/kJkED6bwYAVvImwJtszMVu0BAPAXx1rd1fn1BACA164HHDwF+DpPyKwAAAAASUVORK5CYII=)  
**API-006 — Security misconfiguration**  
| | |  
|-|-|  
|   |   |   
| **Severity** | 🟠 **Medium** |   
| **OWASP API** | API8:2023 – Security Misconfiguration |   
| **CWE** | CWE-16 |   
   
**Sub-findings**  
**(a) Permissive CORS**  
Access-Control-Allow-Origin: *  
 Access-Control-Allow-Credentials: true  
   
This combination is invalid per spec and dangerous in practice — where browsers honour it, any website can make authenticated requests to the API using the visitor's session.  
   
 **Fix:** allow-list specific origins; never reflect the Origin header back; never pair a wildcard with credentials.  
**(b) Verbose error messages**  
{ "error": "SequelizeDatabaseError: column \"usr_email\" does not exist",  
   "stack": "at /app/src/controllers/user.js:47:12" }  
   
This hands an attacker the database schema, the framework, and the file layout.  
   
 **Fix:** return a generic message with a correlation ID; log the detail server-side only.  
**(c) Missing security headers**  
X-Content-Type-Options: nosniff  
 Strict-Transport-Security: max-age=31536000; includeSubDomains  
 Cache-Control: no-store            ← on all responses containing personal data  
 X-Frame-Options: DENY  
   
**(d) Unnecessary HTTP methods enabled**  
curl -s -X OPTIONS -i "$BASE/users/101" | grep -i allow  
 # Allow: GET, POST, PUT, PATCH, DELETE, TRACE   ← TRACE should be disabled  
   
**(e) Documentation or debug interfaces exposed in production**  
   
 Check /swagger-ui.html, /graphql (introspection), /actuator, /debug, /.env, /.git/config.  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANUlEQVR4nO3OMQ2AABAAsSPBCUZfEnoYmFDBhAU2QtIq6DIzW7UHAMBfnGt1V8fXEwAAXrse/wcF74lXkIsAAAAASUVORK5CYII=)  
**API-007 — Improper inventory management**  
| | |  
|-|-|  
|   |   |   
| **Severity** | 🟡 **Low** — but a common root cause of High findings |   
| **OWASP API** | API9:2023 – Improper Inventory Management |   
   
**What it is**  
   
 Old API versions, staging endpoints, or undocumented routes remain reachable from the internet.  
**Why it matters**  
   
 A fix applied to /v2/users does not reach /v1/users. The deprecated version keeps serving the same data with the old vulnerability, and nobody is watching it. Breaches regularly trace back to an endpoint the security team did not know existed.  
**Checks**  
for v in v1 v2 v3 beta internal test dev staging; do  
   printf "%-10s " "$v"  
   curl -s -o /dev/null -w "%{http_code}\n" "$BASE/../$v/users"  
 done  
   
**Remediation**  
1. Maintain a **live API inventory** — every endpoint, its version, owner, environment, and data sensitivity.  
2. Generate documentation from code (OpenAPI) so it cannot drift out of date.  
3. Define a **deprecation policy**: announce, sunset, then actually switch the old version off.  
4. Keep non-production environments off the public internet, and never populate them with real customer data.  
5. Run periodic external discovery scans to find endpoints nobody documented.  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANklEQVR4nO3OQQmAABRAsSfYxZo/kC1sYQLPJrCCNxG2BFtmZquOAAD4i3Ot7mr/egIAwGvXA4qzBdC53Vr8AAAAAElFTkSuQmCC)  
**5. OWASP API Security Top 10 (2023) — Coverage Matrix**  
| | | | |  
|-|-|-|-|  
| **#** | **Risk** | **Tested** | **Result** |   
| API1 | Broken Object Level Authorization | ✅ | [ ] |   
| API2 | Broken Authentication | ✅ | [ ] |   
| API3 | Broken Object Property Level Authorization | ✅ | [ ] |   
| API4 | Unrestricted Resource Consumption | ✅ | [ ] |   
| API5 | Broken Function Level Authorization | ✅ | [ ] |   
| API6 | Unrestricted Access to Sensitive Business Flows | ⬜ | Requires business context |   
| API7 | Server Side Request Forgery | ⬜ | No URL-accepting parameter found |   
| API8 | Security Misconfiguration | ✅ | [ ] |   
| API9 | Improper Inventory Management | ✅ | [ ] |   
| API10 | Unsafe Consumption of APIs | ⬜ | Requires internal architecture visibility |   
   
***API5 (Broken Function Level Authorization)*** * is worth testing explicitly: take an admin-only endpoint such as * *DELETE /users/{id}* * or * *GET /admin/reports* * and call it with a standard user's token. A * *200* * response is a High-severity finding.*  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANUlEQVR4nO3OYQ1AABSAwY9JoICqL4Z8Ikiggn9mu0twy8wc1RkAAH9xbdVa7V9PAAB47X4A9CgEJQFjJ/EAAAAASUVORK5CYII=)  
**6. Remediation Roadmap**  
| | | | | |  
|-|-|-|-|-|  
| **Priority** | **Finding** | **Owner** | **Effort** | **Target** |   
| 1 | API-001 BOLA | Backend | Medium | 7 days |   
| 2 | API-004 Token handling | Backend / Platform | Medium | 7 days |   
| 3 | API-002 Data exposure | Backend | Low | 14 days |   
| 4 | API-005 Mass assignment | Backend | Low | 14 days |   
| 5 | API-003 Rate limiting | Platform / Gateway | Low | 30 days |   
| 6 | API-006 Misconfiguration | DevOps | Low | 30 days |   
| 7 | API-007 Inventory | Architecture | Medium | 60 days |   
   
**Strategic recommendations**  
1. **Centralise authorisation.** Findings API-001 and API-005 are the same root cause — per-endpoint logic that someone forgets on the next endpoint. Move it into middleware.  
2. **Adopt an API gateway** for consistent rate limiting, authentication, and logging.  
3. **Contract-first development.** Define OpenAPI specs, generate validation from them, and reject any request that violates the contract.  
4. **Add authorisation tests to CI.** For every endpoint: unauthenticated → 401; wrong user → 403; correct user → 200. Automated and non-negotiable.  
5. **Log and monitor.** Alert on bursts of 403s (enumeration in progress), unusual per-token volume, and access to newly appearing endpoints.  
6. **Re-test after each release.**  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANklEQVR4nO3OQQmAABRAsSeYxZw/lieLGMACBrCCNxG2BFtmZquOAAD4i3Ot7mr/egIAwGvXA6fGBdgoVMwYAAAAAElFTkSuQmCC)  
**7. Limitations**  
- Black-box assessment; no source code, architecture documentation, or internal network access.  
- Only documented and discoverable endpoints were tested — undiscovered routes may carry further risk.  
- Testing was non-destructive; findings were confirmed to proof-of-existence only.  
- Business-logic abuse (API6) cannot be meaningfully assessed without knowing the intended workflows.  
- Results reflect the API state on [date] only.  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANElEQVR4nO3OQQmAABRAsSdYxKY/jbnMIJ7FCt5E2BJsmZmt2gMA4C+Otbqr8+sJAACvXQ85TgYRMv3/cwAAAABJRU5ErkJggg==)  
**8. References**  
- OWASP API Security Top 10 (2023) — [https://owasp.org/API-Security/editions/2023/en/0x11-t10/](https://owasp.org/API-Security/editions/2023/en/0x11-t10/ "https://owasp.org/API-Security/editions/2023/en/0x11-t10/")  
- OWASP API Security Testing Guide — [https://owasp.org/www-project-api-security/](https://owasp.org/www-project-api-security/ "https://owasp.org/www-project-api-security/")  
- OWASP REST Security Cheat Sheet — [https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html "https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html")  
- OWASP JWT Cheat Sheet — [https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html "https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html")  
- crAPI (vulnerable API for practice) — [https://github.com/OWASP/crAPI](https://github.com/OWASP/crAPI "https://github.com/OWASP/crAPI")  
- NIST SP 800-204 (Microservices security) — [https://csrc.nist.gov/](https://csrc.nist.gov/ "https://csrc.nist.gov/")  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANUlEQVR4nO3OMQ2AABAAsSNhwgJmkPYLLpnRgQU2QtIq6DIze3UGAMBf3Gu1VcfHEQAA3rseaHkEMn1wK7sAAAAASUVORK5CYII=)  
**Appendix A — Repository structure**  
FUTURE_CS_03/  
 ├── README.md  
 ├── API_Security_Risk_Analysis.md  
 ├── API_Security_Risk_Analysis.pdf  
 ├── postman/  
 │   ├── API_Security_Tests.postman_collection.json  
 │   └── API_Security.postman_environment.json     # tokens redacted  
 ├── evidence/  
 │   ├── api-001-bola.png  
 │   ├── api-002-data-exposure.png  
 │   ├── api-003-rate-limit.png  
 │   └── api-004-jwt-decode.png  
 └── scripts/  
     ├── bola_check.sh  
     └── rate_limit_check.sh  
   
**Appendix B — Reusable test scripts**  
#!/usr/bin/env bash  
 # bola_check.sh — verify object-level authorization  
 # Usage: ./bola_check.sh <baseUrl> <tokenA> <userBId>  
 set -euo pipefail  
 BASE="$1"; TOKEN_A="$2"; VICTIM_ID="$3"  
   
 for METHOD in GET PUT PATCH DELETE; do  
   CODE=$(curl -s -o /dev/null -w "%{http_code}" -X "$METHOD" \  
          -H "Authorization: Bearer $TOKEN_A" \  
          -H "Content-Type: application/json" \  
          "$BASE/users/$VICTIM_ID")  
   if [[ "$CODE" == "200" || "$CODE" == "204" ]]; then  
     echo "[VULNERABLE] $METHOD /users/$VICTIM_ID -> $CODE"  
   else  
     echo "[OK]         $METHOD /users/$VICTIM_ID -> $CODE"  
   fi  
 done  
   
#!/usr/bin/env bash  
 # rate_limit_check.sh — detect absence of rate limiting  
 # Usage: ./rate_limit_check.sh <url> <token> [count]  
 set -euo pipefail  
 URL="$1"; TOKEN="$2"; COUNT="${3:-100}"  
 THROTTLED=0  
   
 for _ in $(seq 1 "$COUNT"); do  
   CODE=$(curl -s -o /dev/null -w "%{http_code}" \  
          -H "Authorization: Bearer $TOKEN" "$URL")  
   [[ "$CODE" == "429" ]] && THROTTLED=$((THROTTLED+1))  
 done  
   
 if [[ "$THROTTLED" -eq 0 ]]; then  
   echo "[FINDING] $COUNT requests, zero 429 responses — no rate limiting observed"  
 else  
   echo "[OK] Throttling active after $((COUNT-THROTTLED)) requests"  
 fi  
   
