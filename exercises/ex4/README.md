# Exercise 4 – Software Supply Chain Failures

Vulnerability: [A03:2025 - Software Supply Chain Failures](https://owasp.org/Top10/2025/A03_2025-Software_Supply_Chain_Failures/)

## Table of Contents

- [📖 1. Overview](#1-overview)
- [🚨 2. Vulnerable Code](#2-vulnerable-code)
- [💥 3. Exploitation](#3-exploitation)
- [🛡️ 4. Remediation](#4-remediation)
- [✅ 5. Verification](#5-verification)
- [📌 6. Summary](#6-summary)

---

## 📖 1. Overview

This exercise demonstrates how outdated and vulnerable npm dependencies can silently expose a Node.js CAP application to known, publicly documented exploits — without any flaw in the application's own business logic.
Unlike [SQL Injection]() or [Broken Access Control](), supply chain vulnerabilities do not require the attacker to interact with your application endpoints. The exposure exists as soon as a vulnerable package is installed — in your local workspace, in your [CI/CD pipeline](https://help.sap.com/docs/continuous-integration-and-delivery/sap-continuous-integration-and-delivery/what-is-sap-continuous-integration-and-delivery), or on the [Cloud Foundry runtime](https://help.sap.com/docs/btp/sap-business-technology-platform/development-in-cloud-foundry-environment). 

### 📐 Business Rules
- ❌ Applications must not include [npm packages](https://www.npmjs.com/) with known high or critical CVEs (Common Vulnerabilities and Exposures) in any deployed artifact.

- ⚠️ All direct and transitive dependencies must be continuously inventoried, reviewed, and kept up to date.

- ⚠️ Unmaintained or compromised packages must be replaced or mitigated before any deployment to SAP BTP Cloud Foundry.

### ⚠️ Why This Matters
- **Business Impact:** A single outdated dependency can expose the entire CAP runtime to Server-Side Request Forgery, authentication bypass, or Denial of Service — without exploiting a single line of your custom code.

- **Compliance Risk:** Violates [A03:2025 - Software Supply Chain Failures](https://owasp.org/Top10/2025/A03_2025-Software_Supply_Chain_Failures/) and the [SAP Security Baseline requirement for continuous dependency lifecycle management](https://help.sap.com/docs/btp/sap-btp-security-recommendations-c8a9bb59fe624f0981efa0eff2497d7d/sap-btp-security-recommendations?version=Cloud).

- **Security Risk:** Vulnerable transitive dependencies in package-lock.json are invisible in top-level reviews — and still get deployed to Cloud Foundry.
  
### 🎯 Key Learning Objectives

- **Understand the risk**— Learn how outdated npm packages silently introduce known CVEs into your deployed SAP BTP application artifact, and why keeping dependencies current is a critical line of defense against supply chain attacks.

- **Detect vulnerabilities** — Use npm tooling (npm outdated, npm audit) to inspect the dependency tree, pinpoint version gaps, identify vulnerable packages, and assess their CVE IDs, CVSS scores, and potential real-world impact on your CAP service.

- **Remediate effectively** — Apply safe updates using npm audit fix, npm install @latest, and package.json overrides to handle transitive dependencies that cannot be updated directly.

- **Verify the fix**— Confirm a clean security posture by re-running npm audit and validating a successful cds watch startup with no known vulnerable components remaining.

- **Automate prevention** — Integrate npm audit as an automated build gate in the SAP BTP Continuous Integration & Delivery (CI/CD) pipeline to block vulnerable components from ever reaching production.
  

## 🚨 2. Vulnerable Code

We’ll build upon the same [CAP secure incident management project from the previous exercises](../../README.md#exercises), but this time we focus on the vulnerable dependency set rather than vulnerable application logic.

### What We're Adding

- **`package.json`:** Direct dependencies pinned to vulnerable versions
- **`package-lock.json`:** Resolved tree that may also include vulnerable transitive packages

Below is your `package.json`. For demo, we add the known vulnerable `lodash@4.17.15`, but in real-world you’ll see this same risk as soon as _any_ dependency lags:

```markdown
{
  "name": "incident-management",
  "version": "1.0.0",
  "description": "A simple CAP project.",
  "repository": "<Add your repository here>",
  "license": "UNLICENSED",
  "private": true,
  "dependencies": {
    "@cap-js/hana": "^2",
    "@sap/cds": "~7.9.5",      // ⚠️ Demo risk – exact pin on outdated CAP runtime – behind current supported release, security patches missed!
    "@sap/xssec": "~3.0.0",    // ⚠️ Demo risk – CVE-2023-49583 (CVSS 9.1) – exact pin below 3.6.0 – privilege escalation bypasses XSUAA token validation!
    "express": "4.17.1"        // ⚠️ Demo risk – exact pin on vulnerable release – open redirect + path traversal, fix requires >=4.19.0 (GHSA-rv95-896h)!
  },
  "engines": {
    "node": ">=20"
  },
  "devDependencies": {
    "@cap-js/cds-test": "^0.4.0",
    "@cap-js/cds-types": "^0.11.0",
    "@cap-js/sqlite": "~1.7.0",     // ⚠️ Demo risk – exact pin on compromised release – CVE-2026-46421 (CVSS 9.8) – preinstall hook steals CI/CD secrets!
    "@sap/cds-dk": "^9.1.1",        // ⚠️ Demo risk – pulls transitive js-yaml 4.0.0–4.1.0 – prototype pollution via merge(). [GitHub Advisory GHSA-8j8c-7jfh-h6hx](https://github.com/advisories/GHSA-8j8c-7jfh-h6hx)
    "ui5-task-zipper": "^3.4.2"

... other section objects

```

**Why is this dangerous?**
- `"lodash"` is a public CVE sink—prototype pollution, injection, etc.
- `"express"` and `"@sap/xssec"` have both had vulnerabilities historically.
- No npm audit, SAP Application Vulnerability Report, or blocking gates.

---

## 💥 3. Exploitation

**Step-by-step Attack**

1. **Find Unpatched Package:**  
   Automated scans reveal the version from your app or BTP service instance.
2. **Send Prototype Pollution Payload:**

    ```json
    {
      "title": "CVE exploit",
      "details": { "__proto__": { "polluted": "yes" } }
    }
    ```

3. **App Code (unsafe merge):**

    ```js
    const _ = require('lodash')
    let config = _.merge({}, req.data.details)
    if (config.polluted) {
      // Your prototype is compromised!
    }
    ```

4. **CI/CD and BTP let it ship:**  
   Without blocking on vulnerabilities, exploit code arrives in prod.

---

## 🛡️ 4. Remediation

### a. Add Automated Checks (SAP-native + open source)
- **SAP Application Vulnerability Report:**  
  https://help.sap.com/docs/application-vulnerability-report

- **npm audit**, etc:
    ```sh
    npm install
    npm audit --audit-level=high
    npm outdated
    ```

#### SAP CI/CD YAML Example:
```yaml
steps:
  - script: npm ci
  - script: npm audit --audit-level=high
  - script: npm outdated --long || exit 1
```

### b. Patch All Vulnerable Components

```sh
npm install lodash@4.17.21 --save
```
- Review (`express`, `@sap/xssec`, dev-deps) as well before commit.

### c. Platform Service Check

- In SAP BTP Cockpit:  
    - Check for deprecated/broken service instances (`hana`, `xsuaa`)
- Rerun Application Vulnerability Report after any deployment.

### d. SBOM/Compliance

- Generate SBOM as part of build:
    ```sh
    npx sbom > sbom.json
    ```

---

## ✅ 5. Verification

### Manual

1. Add vulnerable lodash to `package.json`.
2. Run:
    ```sh
    npm audit --audit-level=high
    ```
   Output should flag lodash.
3. Upgrade lodash, rerun, audit is now clean.
4. Your CI/CD should *block* any insecure PR.

### Automated

```sh
npm audit --audit-level=high | grep lodash
npm install lodash@4.17.21 --save
npm audit --audit-level=high | grep lodash # No output = fixed
```

### SAP Dashboard

- Go to BTP Cockpit → Security → Application Vulnerability Report
- Download and check. No vulnerable libraries should report for Incident app.

---

## 📌 6. Summary

**Key outcomes:**
- Attackers exploit slow or ignored dependency updates—this is a practical, not hypothetical, risk.
- SAP AVR + open-source scripting + CI/CD blocks and detects outdated/vulnerable code.
- SBOM generation = regulatory and operational confidence.
- Dependency health must live in build/test, not once-a-year sweeps.
- Train teams to fix *before* deployment—make it routine, not a panic.

**Links for more:**
- [SAP CAP Security Guide](https://cap.cloud.sap/docs/guides/security/)
- [SAP Application Vulnerability Report](https://help.sap.com/docs/application-vulnerability-report)
- [OWASP Top 10, A06](https://owasp.org/Top10/A06_2021-Vulnerable_and_Outdated_Components/)

