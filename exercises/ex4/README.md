# Exercise 4 – Software Supply Chain Failures

Vulnerability: [A03:2025 - Software Supply Chain Failures](https://owasp.org/Top10/2025/A03_2025-Software_Supply_Chain_Failures/)

## Table of Contents

- [📖 1. Overview](#1-overview)
- [🚨 2. Vulnerable Code](#2-vulnerable-code)
- [🔎 3. Detection & Impact Analysis](#3-exploitation) 
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

## 🔎 3.  Detection & Impact Analysis
Before applying any remediation, you must first understand what is vulnerable, why it matters, and what an attacker could realistically achieve in your SAP BTP environment. 
Detection without impact analysis leads to prioritisation errors — patching a moderate [SSRF (Server-Side Request Forgery)](https://cwe.mitre.org/data/definitions/918.html) before a critical authentication bypass is the wrong order.

### Step 1 — Detect the Version Landscape
Run npm outdated to establish the gap between what is installed, what your package.json range version allows, and what the nmp registry currently offers:

'''
npm outdated
'''
You should see the following output :

``` text

secure_incident_management $ npm outdated
Package            Current  Wanted  Latest  Location                        Depended by
@cap-js/cds-test     0.4.1   0.4.1   1.0.1  node_modules/@cap-js/cds-test   secure_incident_management
@cap-js/cds-types   0.11.0  0.11.0  0.17.0  node_modules/@cap-js/cds-types  secure_incident_management
@cap-js/sqlite       1.7.8   1.7.8   2.4.0  node_modules/@cap-js/sqlite     secure_incident_management
@sap/cds             7.9.5   7.9.5   9.9.1  node_modules/@sap/cds           secure_incident_management
@sap/xssec          3.0.10  3.0.10  4.13.0  node_modules/@sap/xssec         secure_incident_management
express             4.17.3  4.17.3   5.2.1  node_modules/express            secure_incident_management

```
### ❌ **Why This Is Vulnerable**

- **The Fingerprint of Exact Pinning (Tilde ~):** Core modules—including @sap/xssec ~3.0.0, @sap/cds ~7.9.5, @cap-js/sqlite ~1.7.0, and express ~4.17.1—are locked below known security fix boundaries. 
Because the 'Current' version equals the 'Wanted' version for every package, running npm update will produce zero changes. The security exposure lies entirely in the 'Current → Latest' gap, which you must close manually.

- **How Tilde Blocks the Patch:**
``` text
  @sap/xssec versions:
    3.0.0  →  3.0.1  →  3.1.0  →  3.2.0  →  3.5.0  →  3.6.0 (CVE fix)
                  ↑ patch           ↑ minor             ↑ minor
  
  ~3.0.0 allows:  ✅ 3.0.1   ❌ 3.1.0   ❌ 3.2.0   ❌ 3.5.0   ❌ 3.6.0
  ^3.0.0 allows:  ✅ 3.0.1   ✅ 3.1.0   ✅ 3.2.0   ✅ 3.5.0   ✅ 3.6.0
```

### Step 2 – Audit for Known Vulnerabilities
Run npm audit to cross-reference every installed package against the npm Advisory Database and produce a CVE-level vulnerability report:

```
npm audit
```
The output below is trimmed to the highest-severity entries — these are the vulnerabilities that require immediate remediation before the service can be safely deployed to SAP BTP Cloud Foundry:

```
$ npm audit

# npm audit report

@sap/xssec  <=3.5.0                                          [🔴 CRITICAL]
Escalation of privileges — authentication bypass via XSUAA JWT validation flaw
Depends on vulnerable versions of: jsonwebtoken, request, requestretry, debug
→ fix: npm audit fix --force  (installs @sap/xssec@3.6.2, outside pinned range)

form-data  <2.5.4                                            [🔴 CRITICAL]
Unsafe random boundary function — transitive via request → requestretry → @sap/xssec
→ fix: npm audit fix --force  (installs @sap/xssec@3.6.2, outside pinned range)

@cap-js/sqlite  1.x                                          [🔴 CRITICAL]
Supply chain compromise via malicious preinstall hook — exfiltrates CI/CD secrets,
XSUAA credentials, and SSH keys at npm install time, before app code executes
→ fix: npm audit fix --force  (installs @cap-js/sqlite@2.4.0, outside pinned range)

... other vulnerabilities 

3 critical vulnerabilities (+ 12 high/moderate/low — run npm audit for full report)

To address all issues (including breaking changes), run:
  npm audit fix --force
```
 #### ❌ **Known vulnerable packages:**

- @sap/xssec ~3.0.0 — CVE-2023-49583 (CVSS 9.1 – 🔴 Critical)

    - The Flaw: Allows unauthenticated attackers to completely bypass XSUAA JWT token validation and forge arbitrary permissions.
    - CAP Impact: Renders all @requires and @restrict security annotations in your CDS service definitions entirely useless in production.

- form-data <2.5.4 — Transitive via @sap/xssec (🔴 Critical)
  - The Flaw: Utilizes an unsafe, predictable random boundary function for multi-part HTTP requests.
  - CAP Impact: Compromises token transmission security inside the core authentication layer. (Note: This package will not appear in your package.json because it is a transitive dependency).

- @cap-js/sqlite ~1.7.0 — CVE-2026-46421 (CVSS 9.8 – 🔴 Critical)
  - The Flaw: A catastrophic supply chain exploit triggered via a malicious preinstall script hook.
  - CAP Impact: Silently steals and exfiltrates your CI/CD secrets, XSUAA client credentials, and local SSH keys the exact millisecond npm install runs—long before any actual application code gets executed.

#### ❌ **The Hidden Supply Chain Risk:**
  To expose exactly how a hidden package entered your project tree, run the dependency lookup command:

  ```
  npm ls form-data
  ```
```
  incident-management (Root)
 └── package.json
      ├── 📦 @cap-js/cds-test@0.4.1 (Direct Dependency)
      │    └── 📦 axios@1.17.0
      │         └── 📦 form-data@4.0.5
      │
      └── 📦 @sap/xssec@3.0.10 (Direct Dependency)
           └── 📦 request@2.88.2
                └── 📦 form-data@2.3.3 [🔴 CRITICAL VULNERABILITY]                   
  ```
⚠️ Critical Hardening Takeaway:

- package.json Blind Spot: Relying solely on a manual review of direct dependencies is insufficient for robust repository hardening.

- Hidden Attack Surface: Transitive dependencies create an unmonitored attack surface by bringing in secondary code that bypasses basic visual checks.

- High-Value Targets: Malicious actors frequently exploit deeply nested utilities because they are rarely audited or updated by application developers.

- Full Privilege Execution: At runtime, these hidden packages execute with the exact same system privileges on SAP BTP as your primary platform libraries.

⚠️ **Critical Hardening Takeaway**
This is why relying solely on a manual review of package.json is insufficient for repository hardening. Transitive dependencies create a hidden attack surface.
Malicious actors frequently target these deeply nested utilities because they are rarely audited or updated by application developers—yet during execution, they run with the exact same system privileges on SAP BTP as your primary platform libraries.

  
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

