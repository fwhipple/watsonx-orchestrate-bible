# Installation

This guide covers the prerequisites, sizing, and installation procedure for deploying watsonx Orchestrate on-premises via IBM Cloud Pak for Data (CPD) on Red Hat OpenShift. SaaS deployments are fully managed by IBM — no installation is required.

!!! note "SaaS deployments"
    If you are using watsonx Orchestrate SaaS, skip this page. All infrastructure is managed by IBM. Proceed to [Architecture](architecture.md) for a conceptual understanding of the platform topology.

---

## Prerequisites

### Red Hat OpenShift Cluster

| Requirement | Minimum | Recommended for Production |
|-------------|---------|---------------------------|
| OpenShift version | 4.14 | 4.15+ |
| Worker nodes | 3 | 5+ |
| Worker node vCPU | 16 per node | 32 per node |
| Worker node RAM | 64 GB per node | 128 GB per node |
| Total cluster RAM | 192 GB | 640 GB+ |

### Storage

Three storage classes are required:

| Storage Type | Purpose | Recommended Solution |
|-------------|---------|---------------------|
| **Block storage** | Databases (PostgreSQL, ClickHouse) | ODF (OpenShift Data Foundation), AWS EBS, Azure Disk |
| **File storage** | Shared config, model artefacts | ODF (CephFS), AWS EFS, Azure Files, NFS |
| **Multi-cloud gateway (object storage)** | Trace archives, long-term data | ODF ObjectBucketClaim, AWS S3, IBM COS |

All storage classes must support the `ReadWriteOnce` access mode for block; `ReadWriteMany` for file storage.

### GPU Requirements (Internal Foundation Model)

If deploying the Internal Foundation Model (IFM) serving stack for on-premises model inference:

| Requirement | Details |
|-------------|---------|
| GPU type | NVIDIA A100 (80 GB) or H100 recommended |
| Minimum GPUs | 2 (for a single replicated model deployment) |
| NVIDIA GPU Operator | Must be installed and configured on the OpenShift cluster |
| CUDA version | 12.x+ |

!!! note "OpenShift AI migration"
    The IFM serving stack is being replaced by Red Hat OpenShift AI (RHOAI) in upcoming releases. Plan GPU scheduling and resource allocation using KServe/RHOAI patterns rather than legacy IFM configuration. See [Platform Roadmap](../reference/roadmap/platform-roadmap.md).

### Private Registry / Air-Gapped Environments

For environments without direct access to the public internet:

1. Set up a private container registry (e.g., IBM Entitled Registry mirror, Red Hat Quay)
2. Mirror all required Orchestrate and CPD operator images to the private registry
3. Configure OpenShift's `ImageContentSourcePolicy` (ICSP) to redirect pulls to the mirror
4. Ensure the OpenShift cluster's pull secret includes credentials for the private registry

---

## Installation Modes

watsonx Orchestrate on CPD supports two deployment modes:

| Mode | Description | Use When |
|------|-------------|----------|
| **Agentic Mode** | Full agent capabilities including multi-agent orchestration, workflows, and advanced LLM routing | Default for new deployments; all sessions in this guide assume Agentic Mode |
| **Agentic Assistant Mode** | Lighter-weight deployment focused on conversational assistant capabilities with reduced resource footprint | Resource-constrained environments; proof-of-concept deployments |

---

## Installation Flow

### Step 1 — Install IBM Cloud Pak for Data

Ensure CPD is installed and healthy on your OpenShift cluster before installing the watsonx Orchestrate cartridge. Follow the [IBM Cloud Pak for Data documentation](https://www.ibm.com/docs/cloud-paks/cp-data) for your target CPD version.

### Step 2 — Install the watsonx Orchestrate Operator

```bash
# Create the dedicated namespace
oc create namespace cpd

# Apply the watsonx Orchestrate subscription via OLM
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: ibm-watson-orchestrate
  namespace: openshift-operators
spec:
  channel: v5.4
  name: ibm-watson-orchestrate
  source: ibm-operator-catalog
  sourceNamespace: openshift-marketplace
EOF
```

### Step 3 — Configure Storage Classes

```yaml
# Example: WatsonxOrchestrate custom resource
apiVersion: orchestrate.watson.ibm.com/v1alpha1
kind: WatsonxOrchestrate
metadata:
  name: watsonxorchestrate
  namespace: cpd
spec:
  version: "5.4.4"
  storageClass:
    block: ocs-storagecluster-ceph-rbd
    file: ocs-storagecluster-cephfs
  licenseAccepted: true
  deploymentMode: agentic
```

### Step 4 — Apply the Custom Resource

```bash
oc apply -f watsonx-orchestrate.yaml
```

Monitor installation progress:

```bash
oc get watsonxorchestrate watsonxorchestrate -n cpd -w
```

Installation typically takes 30–60 minutes depending on cluster performance and image pull speeds.

### Step 5 — Post-Installation Validation

Run the IBM-provided health check script after installation completes:

```bash
# Download and run the health check
curl -O https://github.com/IBM/wxo-health-check/releases/latest/download/wxo-health-check.sh
chmod +x wxo-health-check.sh
./wxo-health-check.sh --namespace cpd
```

A healthy deployment produces output similar to:

```
✓ All Orchestrate pods running
✓ PostgreSQL cluster healthy
✓ ClickHouse cluster healthy
✓ Elasticsearch cluster healthy
✓ Tool Runtime Manager ready
✓ Agent Runtime ready
✓ Observability stack ready
```

See [Troubleshooting](troubleshooting.md) for remediation steps if any checks fail.

---

## Related References

- [Architecture](architecture.md) — Component architecture and topology after installation
- [Troubleshooting](troubleshooting.md) — Post-installation diagnostic tools and common errors
- [Platform Editions](../reference/platform-editions.md) — SaaS vs CPD comparison
- [Platform Roadmap](../reference/roadmap/platform-roadmap.md) — OpenShift AI migration timeline
