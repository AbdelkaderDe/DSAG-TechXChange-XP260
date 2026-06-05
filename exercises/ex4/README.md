# Exercise 4 – Software Supply Chain Failures

Vulnerability: [A03:2025 - Software Supply Chain Failures](https://owasp.org/Top10/2025/A03_2025-Software_Supply_Chain_Failures/)

## Table of Contents

- [📖 1. Overview](#1-overview)
- [🚨 2. Vulnerable Code](#2-vulnerable-code)
- [🔎 3. Vulnerability Analysis](#3-exploitation) 
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

⚠️ Note: Do not copy the code from the **Vulnerable Code** section into your project.

💡 Reminder: Make sure you have opened SAP Business Application Studio (BAS) before starting this exercise. See [Exercise 0, Step 5](../../ex0/README.md#step-5-launch-sap-bas-import-project-and-deploy-to-cloud-foundry)

### What We're Adding

- **`package.json`:** Direct dependencies pinned to vulnerable versions

```markdown
{

> [!WARNING]
> The `package.json` below pins direct dependencies to known-vulnerable versions for demonstration purposes.
> In production, identical risk arises from any dependency — direct or transitive — that lags behind its patched release. 

  "name": "incident-management",
  "version": "1.0.0",
  "description": "A simple CAP project.",
  "repository": "<Add your repository here>",
  "license": "UNLICENSED",
  "private": true,
  "dependencies": {
    "@cap-js/hana": "^2",
    "@sap/cds": "~7.9.5",      // ⚠️ Demo risk – Security patches missed!, exact pin on outdated CAP runtime – behind current supported release, 
    "@sap/xssec": "~3.0.0",    // ⚠️ Demo risk – Privilege escalation bypasses XSUAA token validation! CVE-2023-49583 (CVSS 9.1) – exact pin below 3.6.0 
    "express": "4.17.1"        // ⚠️ Demo risk – Fix requires >=4.19.0,  exact pin on vulnerable release – open redirect + path traversal!
  },
  "engines": {
    "node": ">=20"
  },
  "devDependencies": {
    "@cap-js/cds-test": "^0.4.0",
    "@cap-js/cds-types": "^0.11.0",
    "@cap-js/sqlite": "~1.7.0",     // ⚠️ Demo risk – Preinstall hook steals CI/CD secrets! exact pin on compromised release – CVE-2026-46421 (CVSS 9.8).
    "@sap/cds-dk": "^9.1.1",        // ⚠️ Demo risk – Prototype pollution via merge(). pulls transitive js-yaml 4.0.0–4.1.0
    "ui5-task-zipper": "^3.4.2"

... other section objects

```
- Copy the contents of [package_vulnerable.json](./package_vulnerable.json) into your project’s **package.json** file.
- Open the **SAP BAS Terminal** (`Menu → Terminal → New Terminal`) at the project root and run:

```
# Step 1 — Remove the existing node_modules/ and package-lock.json to force a clean resolution
# Both node_modules/ and package-lock.json must be deleted before installing the vulnerable baseline — otherwise npm resolves from the cached/locked state and never downloads the vulnerable file packages.

rm -rf node_modules package-lock.json

# Step 2 — Install with legacy peer resolution (required by the pinned vulnerable versions)
npm install --legacy-peer-deps

```

You should see the following output confirming the vulnerable packages are installed:

```text
up to date, audited 357 packages in 809ms

42 packages are looking for funding
  run `npm fund` for details

15 vulnerabilities (4 low, 2 moderate, 6 high, 3 critical)

To address issues that do not require attention, run:
  npm audit fix

To address all issues, run:
  npm audit fix --force

Run `npm audit` for details.
```

> [!WARNING]
> npm has detected **15 vulnerabilities across 357 installed packages**, including
> **3 critical** and **6 high** severity findings — all introduced by the pinned
> dependency versions in `package.json`. Do **not** run `npm audit fix --force` yet:
> remediation is covered in the next sections.

> ⚠️ NOTE
> `--legacy-peer-deps` is required because the pinned vulnerable versions conflict
> with current peer dependency requirements. Using this flag in production is not recommended. It simply hides warning messages, meaning you might completely miss serious version conflicts that could break your application..

**Why This Is Vulnerable**
- The vulnerable state is not caused by a single bad package — it is the result of multiple dependency that collectively create an exploitable supply chain exposure  under  [A03:2025 - Software Supply Chain Failures](https://owasp.org/Top10/2025/A03_2025-Software_Supply_Chain_Failures/):

❌ **Exact version pinning (tilde ~):** @sap/xssec ~3.0.0, @sap/cds ~7.9.5, @cap-js/sqlite ~1.7.0, and express 4.17.1 are all pinned using tilde ranges or exact versions that lock your project below known security fix boundaries:
  
  - Missed Patches: * ~3.0.0 resolves only within 3.0.x and will never reach 3.6.0 where CVE-2023-49583 is fixed.
  - 4.17.1 is completely frozen below 4.19.0 where critical open redirect and path traversal fixes live.
  - The Silent Risk: Unlike the caret (^) range, tilde and exact pins cause **'npm install'** to silently skip all minor-version releases—the middle number changes (e.g., 3.0.0 → 3.6.0)—where security fixes are most commonly shipped, without throwing any warnings.
  
  ```
  @sap/xssec versions:
    3.0.0  →  3.0.1  →  3.1.0  →  3.2.0  →  3.5.0  →  3.6.0 (CVE fix)
                  ↑ patch           ↑ minor             ↑ minor
  
  ~3.0.0 allows:  ✅ 3.0.1   ❌ 3.1.0   ❌ 3.2.0   ❌ 3.5.0   ❌ 3.6.0
  ^3.0.0 allows:  ✅ 3.0.1   ✅ 3.1.0   ✅ 3.2.0   ✅ 3.5.0   ✅ 3.6.0
  
  ```
❌ **Known vulnerable packages:**

- @sap/xssec ~3.0.0 — CVE-2023-49583 (CVSS 9.1 – 🔴 Critical)

  - The Flaw: Allows unauthenticated attackers to completely bypass XSUAA JWT token validation and forge arbitrary permissions.
  
  - CAP Impact: Renders all @requires and @restrict security annotations in your CDS service definitions entirely useless in production.

- express 4.17.1 — GHSA-rv95-896h (CVSS 6.1 – 🟡 Medium)

  - The Flaw: Introduces open redirect and path traversal vulnerabilities directly into the HTTP routing layer.
  
  - CAP Impact: Exposes the foundational routing layer that CAP relies on to serve all OData and REST endpoints on Cloud Foundry.

- @cap-js/sqlite ~1.7.0 — CVE-2026-46421 (CVSS 9.8 – 🔴 Critical)

  - The Flaw: A catastrophic supply chain exploit triggered via a malicious preinstall script hook.
  
  - CAP Impact: Silently steals and exfiltrates your CI/CD secrets, XSUAA client credentials, and local SSH keys the exact millisecond npm install runs—long before any actual application code gets executed.
 
❌ **Outdated platform libraries:** 

- @sap/cds ~7.9.5 (Deprecated & Unsupported)
  - Explicitly marked as deprecated and no longer supported by SAP.
  - Pinning to ~7.9.5 means critical security patches, XSUAA compatibility fixes, and hardening improvements shipped in the current ^9 release line will never be applied to this project.

- @cap-js/sqlite ~1.7.0 (Outdated Major Version)
  - Pinned one full major version behind the current ^2 release line.
  - @sap/cds-dk ^9.1.1 (Hidden Transitive Dependency Risk)

- Silently pulls in js-yaml (versions 4.0.0–4.1.0) as a dependency.
  - Carries a prototype pollution vulnerability via the merge() function.
  - The vulnerable package is never visible in package.json, but is present in package-lock.json and deployed to Cloud Foundry on every build.

## 🔎 3.  Vulnerability Analysis

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

