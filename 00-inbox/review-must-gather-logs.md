Objetivo, detectar comando utilizado y errores en la captura.
File: `must-gather.logs`.

```bash title="Image used"
[must-gather      ] OUT 2026-05-22T08:37:37.8834007Z Using must-gather plug-in image: quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:1f471af0848924dae4fd068fd4ae35b6a1bc66e7f27669008c4cc324add27a31
```

```bash title="Check cluster operators"
ClusterOperators:
        clusteroperator/operator-lifecycle-manager is not upgradeable because ClusterServiceVersions blocking minor version upgrades to 4.21.0 or higher:
- maximum supported OCP version for openshift-logging/cluster-logging.v6.2.9 is 4.20
```