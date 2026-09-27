# FUTURE_CS_03 — API Security Risk Analysis

**Internship:** Future Interns — Cyber Security Track
**Task:** 3 of 3

## 📌 Objective
Analyze a REST API for insecure endpoints, authentication/authorization gaps, rate-limiting and
validation weaknesses, and produce a business-friendly risk report with remediation guidance.

## 🎯 Target
[reqres.in](https://reqres.in) — a free public REST API sandbox explicitly built for testing/training
purposes, making it a safe, legal target for this exercise.

## 🛠️ Tools Used
- **Postman** — endpoint testing (auth, payloads, headers)
- **Browser DevTools** — traffic inspection
- **Microsoft Word / Google Docs** — report authoring

## ✅ Key Features Delivered
- Identification of insecure endpoints & data-exposure risks
- Authentication & authorization analysis (tokens, keys, access control)
- Detection of missing rate-limiting / input-validation gaps
- Business-friendly risk explanation, aligned with the OWASP API Security Top 10

## 📊 Summary of Findings

| ID | Finding | Risk |
|----|---------|------|
| A-01 | No authentication required on endpoints returning user data | 🔴 High |
| A-02 | API key passed in URL query parameter instead of header | 🟠 Medium |
| A-03 | No rate limiting on auth/data endpoints | 🟠 Medium |
| A-04 | Minimal input validation on POST endpoints | 🟠 Medium |
| A-05 | Verbose error responses expose stack traces | 🟢 Low |
| A-06 | Overly permissive CORS (`Access-Control-Allow-Origin: *`) | 🟢 Low |

## 📄 Deliverable
[`API_Security_Risk_Analysis_Report.docx`](./API_Security_Risk_Analysis_Report.docx)

## 📁 Repository Structure
```
FUTURE_CS_03/
├── README.md
└── API_Security_Risk_Analysis_Report.docx
```

## 👤 Author
Cyber Security Intern — Future Interns
[LinkedIn](https://www.linkedin.com/company/future-interns/) | contact@futureinterns.com
