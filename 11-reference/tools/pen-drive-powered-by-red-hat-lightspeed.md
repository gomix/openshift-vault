# Pen Drive Powered by Red Hat Lightspeed

## References

- **User Guide:** [Pen Drive powered by Red Hat Lightspeed: User Guide](https://access.redhat.com/articles/7136632)
    
- **Container Image:** [Red Hat Ecosystem Catalog – Pen Drive powered by Red Hat Lightspeed](https://catalog.redhat.com/en/software/containers/pen-drive/pen-drive-scanner-rhel9/68a605de092c681dd3e05d67)
    
- **Release Notes:** [Red Hat Lightspeed Release Notes](https://docs.redhat.com/en/documentation/red_hat_lightspeed/1-latest/html/release_notes/release-notes-january-2026_lightspeed-release-notes)
    

---

## Overview

**Pen Drive Powered by Red Hat Lightspeed** is an offline diagnostic and analysis tool for Red Hat OpenShift. It packages the Red Hat Lightspeed analysis engine into a portable container image that can collect cluster information, analyze existing diagnostic data, and generate health reports without requiring internet access.

The tool is designed for disconnected or security-sensitive environments where cluster data cannot be uploaded to Red Hat services. It complements traditional troubleshooting tools such as **must-gather** and **Red Hat Insights** by providing automated, on-premises analysis and actionable recommendations.

### Typical use cases

- Analyze an existing `must-gather` archive.
    
- Collect and analyze diagnostic data in a single workflow.
    
- Perform health checks on disconnected OpenShift clusters.
    
- Detect configuration drift using **Kube Compare**.
    
- Generate HTML reports for troubleshooting and support.
    

> **Note:** At the time of writing, Pen Drive Powered by Red Hat Lightspeed is available as a **Technology Preview** feature.

## How it works

```
                    +---------------------------+
                    | Laptop / Bastion Host     |
                    |                           |
                    |  Podman                   |
                    |  +---------------------+  |
                    |  | Pen Drive          |  |
                    |  | (container image)  |  |
                    |  +---------------------+  |
                    +-------------+-------------+
                                  |
                           kubeconfig / oc login
                                  |
                                  v
                  +-------------------------------+
                  | OpenShift Cluster             |
                  |                               |
                  | API Server                    |
                  | Insights Operator             |
                  | oc debug (when required)      |
                  | must-gather plugins           |
                  +-------------------------------+
```