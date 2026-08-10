# Phase 1 - Functional Validation (WiP)

## Objective

Validate that the Quay deployment and the benchmark client are fully operational before executing performance tests.

This phase focuses on correctness rather than performance. Any failures detected here must be resolved before proceeding to the benchmark phases.

---

# Scope

The validation verifies the complete path between the benchmark client and Quay, including:

- Network connectivity
- TLS communication
- Authentication
- Repository access
- Artifact upload
- Artifact download
- Artifact integrity

No performance measurements are collected during this phase.

---

# Validation Checklist

| Validation | Expected Result |
|------------|-----------------|
| DNS resolution | Successful |
| TLS handshake | Successful |
| Authentication | Successful |
| Repository exists | Yes |
| Push artifact | Successful |
| Pull artifact | Successful |
| Digest verification | Match |
| Delete artifact (optional) | Successful |

---

# Test Environment

## Registry

- Red Hat Quay
- OpenShift deployment
- TLS enabled
- Robot or user credentials

## Client

- ORAS CLI
- Kubernetes Job
- OpenShift Pod

---

# Test Artifact

A small artifact should be used for validation.

Recommended characteristics:

- Small (< 5 MB)
- Deterministic content
- Fixed digest
- Easy to recreate

Example:

```
hello.txt
```

---

# Validation Steps

## 1. Login

Authenticate against the registry.

Expected result:

- Authentication succeeds.
- Credentials are accepted.

---

## 2. Verify Repository Access

Confirm that the target repository exists and is accessible.

Expected result:

- Repository can be accessed.
- User has push and pull permissions.

---

## 3. Push Artifact

Upload the validation artifact.

Expected result:

- Upload completes successfully.
- Registry returns a valid digest.

---

## 4. Pull Artifact

Download the previously uploaded artifact.

Expected result:

- Download completes successfully.

---

## 5. Verify Integrity

Calculate the digest of the downloaded artifact.

Expected result:

- Local digest matches the registry digest.

---

## 6. Cleanup (Optional)

Remove the validation artifact.

Expected result:

- Repository returns to its initial state.

---

# Success Criteria

The functional validation is considered successful when:

- All operations complete successfully.
- No authentication errors occur.
- No TLS errors occur.
- No authorization errors occur.
- Artifact digest matches after download.
- No unexpected registry errors are reported.

---

# Failure Conditions

Benchmark execution must stop if any of the following occur:

- Authentication failure
- TLS validation failure
- Push failure
- Pull failure
- Digest mismatch
- Registry returns HTTP 5xx errors

---

# Deliverables

Successful completion of this phase confirms that:

- The benchmark client is correctly configured.
- Quay is operational.
- Credentials are valid.
- The repository is usable.
- The environment is ready for performance benchmarking.

The benchmark can then proceed to **Phase 2 – Baseline Performance**.