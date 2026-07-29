---
tags:
  - security
  - cve
  - vulnerability
---

# Evaluating Whether a CVE Applies to Your OpenShift Cluster

## Overview

When a new Common Vulnerabilities and Exposures (CVE) is published, one of the first questions an OpenShift administrator should ask is:

> **Does this CVE affect my OpenShift cluster?**

The answer is not always straightforward.

A CVE may affect multiple Red Hat products, each with its own lifecycle and security advisory. A fix may already be available for one product while still be pending or deferred for another.

Simply finding an OpenShift-related CVE does **not** mean that your OpenShift release already contains a fix.

This article explores how to use the **Red Hat Security Data API** to inspect CVEs associated with OpenShift and understand their current status.

---

# Discovering Recent OpenShift CVEs

The Security Data API can be used to retrieve recently published CVEs associated with OpenShift.

```bash
curl -s \
"https://access.redhat.com/hydra/rest/securitydata/cve.json?created_days_ago=30&product=OpenShift" |
jq -r '
.[] |
[
    .CVE,
    .severity,
    .public_date
] | @tsv
'
```

Example output:

```text
CVE-2026-18107    moderate    2026-07-28T18:21:00Z
CVE-2026-49332    important   2026-07-28T12:00:00Z
CVE-2026-53669    moderate    2026-07-27T22:11:39Z
...
```

This list represents recently published CVEs associated with OpenShift.

However, it does **not** tell you:

- whether OpenShift is affected
- whether a fix already exists
- which OpenShift releases contain the fix

Those questions require inspecting the CVE in more detail.

---

# Understanding `package_state`

Each CVE contains a `package_state` section describing its current status for every affected Red Hat product.

Example:

```json
{
  "product_name": "Red Hat OpenShift Container Platform 4",
  "fix_state": "Fix deferred",
  "package_name": "rhcos"
}
```

Typical values include:

- `Affected`
- `Fix deferred`
- `Fixed`
- `Not affected`
- `Will not fix`

This section answers the question:

> **What is the current status of this CVE for a particular product?**

For example, a status of **Fix deferred** indicates that Red Hat is aware of the issue, but no fix has yet been released for that product.

---

# Understanding `affected_release`

Once a fix has been published, the CVE also contains an `affected_release` section.

Each entry corresponds to a released advisory (typically an RHSA) for a specific Red Hat product.

Unlike `package_state`, this section only contains products for which a fix has already been released.

---

# Inspecting a CVE

After identifying a CVE of interest, retrieve its complete information from the Security Data API.

```bash
CVE=CVE-2026-54370

curl -s \
"https://access.redhat.com/hydra/rest/securitydata/cve/${CVE}.json"
```

To display only the products that already have a published advisory:

```bash
CVE=CVE-2026-54370

curl -s \
"https://access.redhat.com/hydra/rest/securitydata/cve/${CVE}.json" |
jq -r '
    (.affected_release // [])[]
    | "\(.product_name) -> \(.advisory)"
'
```

Example output:

```text
Red Hat Enterprise Linux 10          -> RHSA-2026:42739
Red Hat Enterprise Linux 8           -> RHSA-2026:43420
Red Hat Enterprise Linux 9           -> RHSA-2026:42736
Red Hat Discovery 2                  -> RHSA-2026:46836
Red Hat Hardened Images              -> RHSA-2026:34351
Red Hat Update Infrastructure 5      -> RHSA-2026:44481
```

This output illustrates an important concept:

A single CVE may affect many Red Hat products, and each product receives its own advisory independently.

---

# Example: CVE-2026-53359

The following output was obtained by querying the Security Data API for **CVE-2026-53359**.

```text
Red Hat Enterprise Linux 10                       -> RHSA-2026:36956
Red Hat Enterprise Linux 10                       -> RHSA-2026:43826
Red Hat Enterprise Linux 10.0 Extended Update Support -> RHSA-2026:39371
Red Hat Enterprise Linux 8                        -> RHSA-2026:39082
Red Hat Enterprise Linux 8                        -> RHSA-2026:39083
Red Hat Enterprise Linux 8                        -> RHSA-2026:44004
...
Red Hat OpenShift Container Platform 4.20         -> RHSA-2026:40787
Red Hat OpenShift Container Platform 4.21         -> RHSA-2026:40779
Red Hat OpenShift Container Platform 4.22         -> RHSA-2026:40764
```

This example demonstrates several important characteristics of the Security Data API:

- A single CVE may affect multiple Red Hat products.
- Each product receives its own security advisory.
- Security fixes are released independently for each supported product version.
- In this case, fixes have been released for OpenShift **4.20**, **4.21**, and **4.22**, but **not for 4.18**.

This does **not** necessarily mean that OpenShift 4.18 is vulnerable.

It simply means that no published advisory exists for the 4.18 release stream.

---

# `package_state` vs `affected_release`

These two sections answer different questions.

| Section | Answers |
|----------|---------|
| `package_state` | What is the current status of this CVE for a product? |
| `affected_release` | Has a fix already been released for this product? |

A product may appear in `package_state` with a status of **Fix deferred** while simultaneously being absent from `affected_release`.

Likewise, another Red Hat product may already have a published RHSA for the same CVE.

Understanding the distinction between these two sections is essential when evaluating CVEs.

---

# Key Takeaways

When evaluating a CVE, avoid asking only:

> **Does this CVE affect OpenShift?**

Instead, ask:

> **Has a fix been released for my OpenShift release stream?**

These are fundamentally different questions.

A CVE may already have published fixes for some OpenShift releases while other supported release streams are still waiting for a fix.

Understanding how to interpret both `package_state` and `affected_release` is the first step toward accurately assessing the impact of a CVE on an OpenShift cluster.

---