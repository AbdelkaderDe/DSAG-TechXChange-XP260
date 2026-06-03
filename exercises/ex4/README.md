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
- ❌ Applications must not include npm packages with known high or critical CVEs (Common Vulnerabilities and Exposures) in any deployed artifact.

- ⚠️ All direct and transitive dependencies must be continuously inventoried, reviewed, and kept up to date.

- ⚠️ Unmaintained or compromised packages must be replaced or mitigated before any deployment to SAP BTP Cloud Foundry.

### ⚠️ Why This Matters
- **Business Impact:** A single outdated dependency can expose the entire CAP runtime to Server-Side Request Forgery, authentication bypass, or Denial of Service — without exploiting a single line of your custom code.

- **Compliance Risk:** Violates [A03:2025 - Software Supply Chain Failures](https://owasp.org/Top10/2025/A03_2025-Software_Supply_Chain_Failures/), the [SAP Security Baseline requirement for continuous dependency lifecycle management](https://help.sap.com/docs/btp/sap-btp-security-recommendations-c8a9bb59fe624f0981efa0eff2497d7d/sap-btp-security-recommendations?version=Cloud).

- **Security Risk:** Vulnerable transitive packages — those not listed in package.json directly but present in package-lock.json and deployed to Cloud Foundry — are especially dangerous because they are invisible to developers who only inspect top-level dependencies.

### 🎯 Key Learning Objectives
- Understand how outdated npm packages introduce known CVEs into the deployed SAP BTP application artifact.

- Use npm outdated to identify version gaps between installed packages and their latest safe releases.

- Use npm audit to map each vulnerable package to its CVE ID, CVSS score, and real-world impact on the CAP service.

- Remediate vulnerabilities using npm audit fix, npm install <pkg>@latest, and package.json overrides for transitive dependencies.

- Verify the remediated state by re-running npm audit and confirming a clean cds watch startup.

- Integrate npm audit as an automated build gate in the SAP BTP Continuous Integration & Delivery pipeline to prevent vulnerable components from reaching production.



This exercise demonstrates how outdated and vulnerable npm packages can expose a Node.js CAP application to known security flaws. 

In the incident management application from the previous exercises, 
the `package.json` file pins several dependencies to versions with publicly known CVEs. Because these packages are deployed together with the application, every vulnerable component becomes part of the production attack surface.

In this exercise you will test the application locally in the development environment. Instead of building and deploying the application to SAP BTP, you will inspect the dependency tree, identify vulnerable packages with npm tooling, apply safe updates, and verify that the project no longer contains known vulnerable components.

Vulnerable and outdated components are a top supply chain risk for all Node.js and SAP CAP apps. If both application and BTP service dependencies aren’t checked and updated continuously, attackers will exploit them—regardless of business logic security.

This lab is hands-on, using your real CAP project structure and pipelines.

---

## 🚨 2. Vulnerable Code

Below is your `package.json`. For demo, we add the known vulnerable `lodash@4.17.15`, but in real-world you’ll see this same risk as soon as _any_ dependency lags:

```json
{
  "name": "incident-management",
  "version": "1.0.0",
  "description": "A simple CAP project.",
  "dependencies": {
    "@cap-js/hana": "^2",
    "@sap/cds": "^9",
    "@sap/xssec": "^4.8.0",
    "express": "^4",
    "lodash": "4.17.15" // ⚠️ Demo risk – real CVEs for <4.17.21!
  },
  "engines": {
    "node": ">=20"
  },
  "devDependencies": {
    "@cap-js/cds-test": "^0.4.0",
    "@cap-js/cds-types": "^0.11.0",
    "@cap-js/sqlite": "^2",
    "@sap/cds-dk": "^9.1.1",
    "ui5-task-zipper": "^3.4.2"
  }
}
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

