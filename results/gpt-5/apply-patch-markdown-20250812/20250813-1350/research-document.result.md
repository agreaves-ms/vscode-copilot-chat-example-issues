<!-- markdownlint-disable-file -->
# Task Research Documents: AzureML Edge Arc Integration

🎯 Comprehensive research to design a new Terraform-only edge component that integrates Azure Machine Learning with Arc-enabled Kubernetes. The component must mirror the cloud component structure, store AzureML SSL material as Key Vault secrets (in the edge component), and rely on Kubernetes resources (namespace, SecretProviderClass, SecretSync) applied by the IoT Operations edge component.

## 📋 Policy Framework

- **SSE-Only Policy**: This solution standardizes on the Azure Key Vault Secret Store extension (SSE) for secret synchronization. Other approaches (e.g., online-only AKV Secrets Provider) are out of scope.
- **Private-Only Connectivity**: Only private VNet connectivity is supported. Public internet access to the AzureML workspace or Arc-exposed services (including `public_network_access_enabled = true` or LoadBalancer endpoints) is explicitly NOT supported. All endpoint exposure patterns MUST remain within private address spaces reachable via same VNet, peering, hub-spoke, or approved private connectivity (VPN/ExpressRoute) only.
- **Security-First Design**: No secret values in Terraform outputs; TLS enabled by default; strict private connectivity; NodePort restricted via private NSGs/firewalls.

## Table of Contents

- [Task Research Documents: AzureML Edge Arc Integration](#task-research-documents-azureml-edge-arc-integration)
  - [📋 Policy Framework](#-policy-framework)
  - [Table of Contents](#table-of-contents)
  - [Outline](#outline)
  - [Scope and Success Criteria](#scope-and-success-criteria)
  - [🎯 Recommended Technical Solution](#-recommended-technical-solution)
  - [🔑 Important Discoveries](#-important-discoveries)
    - [✅ Validated Azure Provider Capabilities](#-validated-azure-provider-capabilities)
    - [🏗️ Component Architecture Alignment](#️-component-architecture-alignment)
    - [🔐 SSL and Kubernetes Integration (SSE-Only)](#-ssl-and-kubernetes-integration-sse-only)
    - [🛡️ Private VNet Requirements](#️-private-vnet-requirements)
  - [1 - 🔑 Key Discoveries](#1----key-discoveries)
    - [✅ Validated Azure Provider Capabilities](#-validated-azure-provider-capabilities-1)
    - [🏗️ Component Architecture Alignment](#️-component-architecture-alignment-1)
    - [🔐 SSL and Kubernetes Integration (SSE-Only)](#-ssl-and-kubernetes-integration-sse-only-1)
    - [🛡️ Private VNet Requirements](#️-private-vnet-requirements-1)
  - [Research Executed](#research-executed)
    - [🔍 Technical Deep-Dive Validation (2025-08-10)](#-technical-deep-dive-validation-2025-08-10)
    - [📋 Project Structure Validation](#-project-structure-validation)
    - [🔗 IoT Operations Integration Requirements](#-iot-operations-integration-requirements)
    - [🛡️ Network Security Policy Enforcement](#️-network-security-policy-enforcement)
    - [📁 File Analysis Evidence](#-file-analysis-evidence)
    - [🔎 Code Search Results](#-code-search-results)
    - [📄 External Research Evidence Log](#-external-research-evidence-log)
  - [🔑 Original Discoveries](#-original-discoveries)
    - [✅ Validated Azure Provider Capabilities](#-validated-azure-provider-capabilities-2)
    - [🏗️ Component Architecture Alignment](#️-component-architecture-alignment-2)
    - [Additional Thoughts](#additional-thoughts)
    - [🔐 SSL and Kubernetes Integration (SSE-Only)](#-ssl-and-kubernetes-integration-sse-only-2)
    - [🛡️ Private VNet Requirements](#️-private-vnet-requirements-2)
  - [2222 - 🔑 Key Discoveries](#2222----key-discoveries)
    - [✅ Validated Azure Provider Capabilities](#-validated-azure-provider-capabilities-3)
    - [🏗️ Component Architecture Alignment](#️-component-architecture-alignment-3)
    - [Additional Thoughts](#additional-thoughts-1)
    - [🔐 SSL and Kubernetes Integration (SSE-Only)](#-ssl-and-kubernetes-integration-sse-only-3)
    - [🛡️ Private VNet Requirements](#️-private-vnet-requirements-3)
  - [3333 - 🔑 Key Discoveries](#3333----key-discoveries)
    - [✅ Validated Azure Provider Capabilities](#-validated-azure-provider-capabilities-4)
    - [🏗️ Component Architecture Alignment](#️-component-architecture-alignment-4)
    - [Additional Thoughts](#additional-thoughts-2)
    - [🔐 SSL and Kubernetes Integration (SSE-Only)](#-ssl-and-kubernetes-integration-sse-only-4)
    - [🛡️ Private VNet Requirements](#️-private-vnet-requirements-4)

## Outline

🎯 **Research Scope**: Design and validate a new Terraform-only edge component `src/100-edge/140-azureml/terraform` that integrates Azure Machine Learning with Arc-enabled Kubernetes, following established project patterns.

🏗️ **Technical Architecture Validated**:
- Arc extension resource (`azurerm_arc_kubernetes_cluster_extension`) with `Microsoft.AzureML.Kubernetes` type
- Workspace attachment (`azurerm_machine_learning_inference_cluster`) for Arc cluster registration
- SSL/TLS flow: Key Vault → SSE → Kubernetes Secret → AML Extension configuration
- Component structure mirroring cloud AzureML component with edge-specific modules

🔑 **Key Findings**:
- All API versions and configuration keys confirmed from authoritative Microsoft Learn sources
- SSE CRDs and secret synchronization patterns established and validated
- Private VNet-only configuration requirements documented (public access excluded from scope)
- Security-first defaults with private connectivity and NodePort service restriction

✅ **Implementation Readiness**:
- Dependencies identified and validated against existing components
- Complete file structure and variable organization designed
- Integration scripts pattern established for IoT Operations coordination
- Network security options evaluated and selected

## Scope and Success Criteria

- **Scope**: Design a new Terraform edge component integrating Azure Machine Learning with Arc-enabled Kubernetes, including SSL management via Azure Key Vault and SSE, following established project patterns and conventions
- **Exclusions**: Implementation/scaffolding (implementation planning only), alternative secret sync approaches (SSE-only policy), non-Terraform solutions
- **Assumptions**:
  - Arc cluster has OIDC issuer and workload identity enabled (provided by IoT Operations)
  - Azure Key Vault and AML workspace exist as dependencies
  - Private-only connectivity enforced: cloud workspace and Arc cluster communicate exclusively over private networks (same VNet, peered VNets, hub-spoke, or secured hybrid link)
- **Success Criteria**:
  - ✅ Technical validation of all Azure resources and API versions
  - ✅ Complete component structure design following project conventions
  - ✅ SSL/TLS flow documented with SSE integration pattern
  - ✅ Private VNet-only configuration requirements established (public scenarios excluded)
  - ✅ Implementation guidance with specific file structures and dependencies
  - ✅ Network security patterns evaluated and selected
  - ✅ All technical details backed by authoritative sources with proper references

## 🎯 Recommended Technical Solution

**Selected Architecture**: Create new component `src/100-edge/140-azureml/terraform` following established edge component patterns, with two internal modules and CI wrapper. Store SSL certificates in Key Vault via the 140-azureml component; deploy Kubernetes resources for azureml namespace via 110-iot-ops apply scripts (ServiceAccount, SecretProviderClass, SecretSync). Configure the AML extension with NodePort and TLS using SSE-synced secrets in private VNet scenarios only (no public exposure).

**Key Design Decisions**:
1. **SSE-Only Policy**: Standardize on Secret Store Extension vs. online-only AKV Secrets Provider for consistency with edge/offline scenarios
2. **Private-Only Connectivity**: Public network access for the AML workspace and inference endpoints is not supported; `public_network_access_enabled` remains `false` always; exposure via internal private networking only
3. **Component Separation**: Clear boundaries between Terraform (140-azureml) and Kubernetes manifests (110-iot-ops)
4. **Security-First**: No secret values in Terraform outputs; TLS enabled by default; strict private connectivity; NodePort restricted via private NSGs/firewalls
5. **Operational Simplicity**: Self-signed certificates supported for non-production inside private environments; production requires managed or externally provided certificates; automated sync via SSE

**Implementation Readiness**: All technical dependencies validated, API versions confirmed, and integration patterns established from existing components.

## 🔑 Important Discoveries

### ✅ Validated Azure Provider Capabilities
- **Arc Extension Deployment**: Use `azurerm_arc_kubernetes_cluster_extension` with `extension_type = "Microsoft.AzureML.Kubernetes"`, `identity { type = "SystemAssigned" }`, and `configuration_settings` for all AzureML configuration
- **Compute Attachment**: `azurerm_machine_learning_inference_cluster` attaches Arc cluster to workspace using `kubernetes_cluster_id`; SSL handled at extension level, not attachment level

### 🏗️ Component Architecture Alignment
- **Edge Component Structure**: `src/100-edge/140-azureml/terraform` mirrors cloud `000-cloud/080-azureml` with two internal modules:
  - `modules/azureml-extension-arc`: Encapsulates `azurerm_arc_kubernetes_cluster_extension`
  - `modules/compute-target-attachment`: Encapsulates `azurerm_machine_learning_inference_cluster`
- **Root Module**: Handles naming locals, optional TLS generation, Key Vault secret creation, module orchestration, and outputs

### 🔐 SSL and Kubernetes Integration (SSE-Only)
- **Edge Component (140-azureml)**: Owns TLS materials generation (optional) and Key Vault secret storage (cert/key)
- **IoT Ops (110-iot-ops)**: Owns Kubernetes manifests via apply scripts (azureml namespace, ServiceAccount, SecretProviderClass, SecretSync)
- **Extension Integration**: Uses `sslSecret` and `sslCname` configuration settings for TLS-enabled scenarios

### 🛡️ Private VNet Requirements
- **Cloud Workspace**: `public_network_access_enabled = false` enforced for edge scenarios
- **Network Isolation**: Support for `managed_network.isolation_mode = "AllowOnlyApprovedOutbound"`
- **Private Endpoints**: DNS zones `privatelink.api.azureml.ms` and `privatelink.notebooks.azure.net` for commercial cloud

## 1 - 🔑 Key Discoveries

### ✅ Validated Azure Provider Capabilities
- **Arc Extension Deployment**: Use `azurerm_arc_kubernetes_cluster_extension` with `extension_type = "Microsoft.AzureML.Kubernetes"`, `identity { type = "SystemAssigned" }`, and `configuration_settings` for all AzureML configuration
- **Compute Attachment**: `azurerm_machine_learning_inference_cluster` attaches Arc cluster to workspace using `kubernetes_cluster_id`; SSL handled at extension level, not attachment level

### 🏗️ Component Architecture Alignment
- **Edge Component Structure**: `src/100-edge/140-azureml/terraform` mirrors cloud `000-cloud/080-azureml` with two internal modules:
  - `modules/azureml-extension-arc`: Encapsulates `azurerm_arc_kubernetes_cluster_extension`
  - `modules/compute-target-attachment`: Encapsulates `azurerm_machine_learning_inference_cluster`
    - **Root Module (1)**: Handles naming locals, optional TLS generation, Key Vault secret creation, module orchestration, and outputs
      - **Root Module (2)**: Handles naming locals, optional TLS generation, Key Vault secret creation, module orchestration, and outputs
- **Root Module (3)**: Handles naming locals, optional TLS generation, Key Vault secret creation, module orchestration, and outputs

### 🔐 SSL and Kubernetes Integration (SSE-Only)
- **Edge Component (140-azureml)**: Owns TLS materials generation (optional) and Key Vault secret storage (cert/key)
- **IoT Ops (110-iot-ops)**: Owns Kubernetes manifests via apply scripts (azureml namespace, ServiceAccount, SecretProviderClass, SecretSync)
- **Extension Integration**: Uses `sslSecret` and `sslCname` configuration settings for TLS-enabled scenarios

### 🛡️ Private VNet Requirements
- **Cloud Workspace**: `public_network_access_enabled = false` enforced for edge scenarios
- **Network Isolation**: Support for `managed_network.isolation_mode = "AllowOnlyApprovedOutbound"`
- **Private Endpoints**: DNS zones `privatelink.api.azureml.ms` and `privatelink.notebooks.azure.net` for commercial cloud

## Research Executed

### 🔍 Technical Deep-Dive Validation (2025-08-10)

**AzureML Extension Configuration Matrix** (validated from Microsoft Learn):
- **Extension Configuration Complete**: All configuration keys confirmed from authoritative Microsoft Learn sources
  - `enableTraining` and `enableInference`: **Required** - Must be set to `True` explicitly for respective workloads
  - `inferenceRouterServiceType`: **Required when enableInference=True** - Valid values: `LoadBalancer`, `NodePort`, `ClusterIP`
  - `allowInsecureConnections`: Optional, default `False` - Set to `True` only for dev/test HTTP scenarios
  - `sslSecret`: **Required when allowInsecureConnections=False** - Must reference Kubernetes Secret name in `azureml` namespace
  - `sslCname`: **Required when allowInsecureConnections=False** - TLS/SSL CNAME for HTTPS endpoint
  - `nodeSelector.*`: Optional - Format `nodeSelector.key1=value1` for workload placement restrictions

**Terraform Provider Resource Validation** (Provider 4.39.0):
- **azurerm_arc_kubernetes_cluster_extension**: API Microsoft.KubernetesConfiguration 2024-11-01
- **azurerm_machine_learning_inference_cluster**: API Microsoft.ContainerService 2025-02-01, Microsoft.MachineLearningServices 2025-06-01
  - Purpose: Attaches existing Kubernetes cluster to ML workspace (does NOT create cluster)
  - For Arc clusters: Use `kubernetes_cluster_id` of Arc-connected cluster resource
  - SSL handling: `ssl` block is AKS-focused; for Arc, TLS handled at extension level
- **azurerm_machine_learning_workspace**: API Microsoft.MachineLearningServices 2025-06-01

**Secret Store Extension (SSE) Validation**:
- Extension type: `microsoft.azure.secretstore`
- Two CRDs: `SecretProviderClass` and `SecretSync`
- Secret shape: Opaque type with `cert.pem` and `key.pem` keys in `azureml` namespace
- SecretSync creates Kubernetes secret automatically when using SSE

### 📋 Project Structure Validation

**Component Organization** (validated against existing patterns):
- **Edge Component**: `src/100-edge/140-azureml/` follows established patterns from 110-iot-ops and 120-observability
- **Naming Convention**: Decimal sequence (140 for model management deployment order)
- **Module Structure**: Two internal modules required: `azureml-extension-arc` and `compute-target-attachment`
- **Apply Scripts Integration**: IoT Ops pattern extended for AzureML namespace and SSE resources

**Variable Organization Patterns** (from existing edge components):
- `variables.deps.tf`: Required dependency objects with type definitions
- `variables.core.tf`: Core naming and instance configuration
- `variables.flags.tf`: Feature flags (SSL, training, inference)
- `variables.ext.tf`: Extension configuration (service type, HA, version, release train)

### 🔗 IoT Operations Integration Requirements

**Apply Scripts Enhancement**:
- New script required for azureml namespace, ServiceAccount, SecretProviderClass, and SecretSync
- Template from existing `apply-trust.sh` pattern with kubectl and envsubst
- Environment variables: `TF_SSE_USER_ASSIGNED_CLIENT_ID`, `TF_KEY_VAULT_NAME`, `TF_AZURE_TENANT_ID`

**Federated Identity Credentials**:
- Existing SSE UAMI needs additional FIC for `azureml-ssc-sa` in `azureml` namespace
- Subject format: `system:serviceaccount:azureml:azureml-ssc-sa`
- Pattern follows existing workload identity integration in IoT Operations component

### 🛡️ Network Security Policy Enforcement

**Private-Only Connectivity** (policy confirmed):
- `inferenceRouterServiceType` restricted to `NodePort` only (validation enforced)
- No `LoadBalancer` support to prevent accidental public exposure
- AML workspace `public_network_access_enabled = false` (hard requirement)
- All endpoint access via private networking (VNet, peering, VPN/ExpressRoute only)

### 📁 File Analysis Evidence
- `src/100-edge/110-iot-ops/terraform/modules/iot-ops-init/main.tf`
  - Uses multiple `azurerm_arc_kubernetes_cluster_extension` resources with SystemAssigned identity and configuration settings, confirming pattern for Arc extensions (L24–L38, L40–L57, L59–L70, L72–L86)
- `src/100-edge/120-observability/terraform/modules/cluster-extensions-obs/main.tf`
  - Manages `azurerm_arc_kubernetes_cluster_extension` with `configuration_settings`, providing template for extension module structure
- `src/000-cloud/080-azureml/terraform/modules/inference-cluster-integration/main.tf`
  - Implements `azurerm_machine_learning_inference_cluster` for AKS attach; SSL block uses leaf-domain pattern for AKS (L13–L25)
- `src/000-cloud/080-azureml/terraform/modules/workspace/main.tf`
  - AML workspace sets `public_network_access_enabled = var.public_network_access_enabled` with identity (L17–L29)

### 🔎 Code Search Results
- **Arc Extensions**: Used extensively in edge components (110-iot-ops, 120-observability) with consistent patterns
- **ML Inference Cluster**: Present in cloud AzureML module with AKS-focused SSL options
- **ML Workspace**: Provider supports private networking configuration options

### 📄 External Research Evidence Log

**AzureRM Provider Documentation**:
- **Arc Extension Resource**: Configuration settings, identity requirements, API provider Microsoft.KubernetesConfiguration 2024-11-01
- **ML Inference Cluster**: Attach semantics for existing clusters, API providers Microsoft.ContainerService 2025-02-01, Microsoft.MachineLearningServices 2025-06-01
- **ML Workspace**: Private networking options, managed network isolation modes, API provider Microsoft.MachineLearningServices 2025-06-01

**Microsoft Learn Documentation**:
- **SSE Configuration**: SecretProviderClass and SecretSync CRD creation patterns, troubleshooting guidance
- **AML Extension TLS**: Required secret shape (Opaque type with cert.pem/key.pem keys in azureml namespace)
- **Workload Identity**: ServiceAccount annotation patterns and federated identity credential subject formats
- **Private Networking**: Private endpoint subresources and DNS zone requirements (privatelink.api.azureml.ms)

**Project Conventions Validation**:
- Terraform and shell script standards from `.github/instructions/` applied
- Component structure patterns validated against existing edge components
- Variable organization following established 100-edge component conventions

## 🔑 Original Discoveries

### ✅ Validated Azure Provider Capabilities
- **Arc Extension Deployment**: Use `azurerm_arc_kubernetes_cluster_extension` with `extension_type = "Microsoft.AzureML.Kubernetes"`, `identity { type = "SystemAssigned" }`, and `configuration_settings` for all AzureML configuration
- **Compute Attachment**: `azurerm_machine_learning_inference_cluster` attaches Arc cluster to workspace using `kubernetes_cluster_id`; SSL handled at extension level, not attachment level

### 🏗️ Component Architecture Alignment
- **Edge Component Structure**: `src/100-edge/140-azureml/terraform` mirrors cloud `000-cloud/080-azureml` with two internal modules:
  - `modules/azureml-extension-arc`: Encapsulates `azurerm_arc_kubernetes_cluster_extension`
  - `modules/compute-target-attachment`: Encapsulates `azurerm_machine_learning_inference_cluster`
    - **Root Module**: Handles naming locals, optional TLS generation, Key Vault secret creation, module orchestration, and outputs
      - **Root Module**: Handles naming locals, optional TLS generation, Key Vault secret creation, module orchestration, and outputs
- **Root Module**: Handles naming locals, optional TLS generation, Key Vault secret creation, module orchestration, and outputs

### Additional Thoughts

These forward-looking considerations focus on Day-2 reliability, security reinforcement, and operational simplicity specific to the edge AzureML extension scenario (distinct from earlier 1 / 2222 phases):

1. Secret Lifecycle & Rotation
  - Establish rotation cadence (e.g., 90 days) for self-signed certs; pipeline job: terraform taint cert module → apply → verify SSE secret hash change.
  - Future: optional AKV Certificate object (vs discrete secrets) once provider supports streamlined export for edge consumption.
  - Drift probe: compare SHA256 of Kubernetes secret cert against latest Key Vault version; alert if mismatch persists > grace window.

2. Extension Configuration Guardrails
  - Terraform validation to constrain `inferenceRouterServiceType` = `NodePort` (policy enforcement for private-only posture).
  - Minimal variable surface: expose only version pin, enableInference, enableTraining, ssl flags; hide advanced toggles until justified.
  - Version pin with documented upgrade + rollback path (force replace if schema drift encountered).

3. Failure Mode Mitigations
  - Secret sync lag: pre-flight wait (exponential backoff) for secret keys before creating extension.
  - Partial apply avoidance: explicit depends_on chain + readiness script gating.
  - Private DNS misconfig: helper script resolves AML endpoints, asserts RFC1918 / approved CIDR before success mark.

4. Observability Enhancements
  - Label extension pods (`component=azureml-edge`) for targeted log & metric aggregation in 120-observability.
  - Terraform output (non-sensitive) of Key Vault secret version used → aids audit traceability.
  - Post-deploy synthetic HTTPS probe (curl NodePort) validating 200 + CN match + latency baseline capture.

5. Security Reinforcements
  - Namespace NetworkPolicy: default deny + scoped egress (observability + control plane) once IoT Ops seeds policies.
  - Ensure no legacy service account token reliance (workload identity only) reducing credential sprawl.
  - Plan migration to RBAC-only Key Vault (remove explicit access policies) when provider parity fully stable.

6. Operational Runbooks (Seed)
  - Rotation: regen → KV update → SSE propagation check (hash) → restart router pod if no live reload.
  - Upgrade: bump version → plan diff review → canary edge site → phased rollout → fleet completion.
  - Rollback: revert version var → apply (replace) → validate health + synthetic probe.

7. Testing Layers
  - Static: terraform validate + custom policy checks (serviceType, public access flags, identity presence).
  - Template lint: CI renders Kubernetes manifest templates (envsubst dry-run) verifying no placeholder leakage.
  - Conformance: post-apply script asserts (a) secret keys present, (b) extension Healthy, (c) NodePort unreachable externally.

8. Drift & Compliance Monitoring
  - Scheduled read-only terraform plan; publish diff summary for security dashboard.
  - Store canonical hash of `configuration_settings` JSON; recompute per apply to catch silent portal edits.

9. Backlog / Enhancements
  - Managed cert acquisition (ACME private CA) for large fleets.
  - Zero-downtime extension upgrade (staggered pod restart) if future dual-version support emerges.
  - OPA/Gatekeeper policies enforcing NodePort + namespace isolation invariants.

10. Assumptions Requiring Validation Pre-GA
  - SSE propagation P95 latency < 2 minutes (empirical measurement phase).
  - NodePort strictly private (validated via controlled external scan in staging).
  - All extension cloud calls satisfied via workload identity (no fallback secret mounts found in logs).

Summary: Feasibility confirmed; focus shifts to codifying guardrails (validation + drift detection), automating cert lifecycle, and embedding health probes for low-touch Day-2 operations.

### 🔐 SSL and Kubernetes Integration (SSE-Only)
- **Edge Component (140-azureml)**: Owns TLS materials generation (optional) and Key Vault secret storage (cert/key)
- **IoT Ops (110-iot-ops)**: Owns Kubernetes manifests via apply scripts (azureml namespace, ServiceAccount, SecretProviderClass, SecretSync)
- **Extension Integration**: Uses `sslSecret` and `sslCname` configuration settings for TLS-enabled scenarios

### 🛡️ Private VNet Requirements
- **Cloud Workspace**: `public_network_access_enabled = false` enforced for edge scenarios
- **Network Isolation**: Support for `managed_network.isolation_mode = "AllowOnlyApprovedOutbound"`
- **Private Endpoints**: DNS zones `privatelink.api.azureml.ms` and `privatelink.notebooks.azure.net` for commercial cloud


## 2222 - 🔑 Key Discoveries

### ✅ Validated Azure Provider Capabilities
- **Arc Extension Deployment**: Use `azurerm_arc_kubernetes_cluster_extension` with `extension_type = "Microsoft.AzureML.Kubernetes"`, `identity { type = "SystemAssigned" }`, and `configuration_settings` for all AzureML configuration
- **Compute Attachment**: `azurerm_machine_learning_inference_cluster` attaches Arc cluster to workspace using `kubernetes_cluster_id`; SSL handled at extension level, not attachment level

### 🏗️ Component Architecture Alignment
- **Edge Component Structure**: `src/100-edge/140-azureml/terraform` mirrors cloud `000-cloud/080-azureml` with two internal modules:
  - `modules/azureml-extension-arc`: Encapsulates `azurerm_arc_kubernetes_cluster_extension`
  - `modules/compute-target-attachment`: Encapsulates `azurerm_machine_learning_inference_cluster`
    - **Root Module**: Handles naming locals, optional TLS generation, Key Vault secret creation, module orchestration, and outputs
      - **Root Module**: Handles naming locals, optional TLS generation, Key Vault secret creation, module orchestration, and outputs
- **Root Module**: Handles naming locals, optional TLS generation, Key Vault secret creation, module orchestration, and outputs

### Additional Thoughts

### 🔐 SSL and Kubernetes Integration (SSE-Only)
- **Edge Component (140-azureml)**: Owns TLS materials generation (optional) and Key Vault secret storage (cert/key)
- **IoT Ops (110-iot-ops)**: Owns Kubernetes manifests via apply scripts (azureml namespace, ServiceAccount, SecretProviderClass, SecretSync)
- **Extension Integration**: Uses `sslSecret` and `sslCname` configuration settings for TLS-enabled scenarios

### 🛡️ Private VNet Requirements
- **Cloud Workspace**: `public_network_access_enabled = false` enforced for edge scenarios
- **Network Isolation**: Support for `managed_network.isolation_mode = "AllowOnlyApprovedOutbound"`
- **Private Endpoints**: DNS zones `privatelink.api.azureml.ms` and `privatelink.notebooks.azure.net` for commercial cloud

## 3333 - 🔑 Key Discoveries

### ✅ Validated Azure Provider Capabilities
- **Arc Extension Deployment**: Use `azurerm_arc_kubernetes_cluster_extension` with `extension_type = "Microsoft.AzureML.Kubernetes"`, `identity { type = "SystemAssigned" }`, and `configuration_settings` for all AzureML configuration
- **Compute Attachment**: `azurerm_machine_learning_inference_cluster` attaches Arc cluster to workspace using `kubernetes_cluster_id`; SSL handled at extension level, not attachment level

### 🏗️ Component Architecture Alignment
- **Edge Component Structure**: `src/100-edge/140-azureml/terraform` mirrors cloud `000-cloud/080-azureml` with two internal modules:
  - `modules/azureml-extension-arc`: Encapsulates `azurerm_arc_kubernetes_cluster_extension`
  - `modules/compute-target-attachment`: Encapsulates `azurerm_machine_learning_inference_cluster`
    - **Root Module**: Handles naming locals, optional TLS generation, Key Vault secret creation, module orchestration, and outputs
      - **Root Module**: Handles naming locals, optional TLS generation, Key Vault secret creation, module orchestration, and outputs
- **Root Module**: Handles naming locals, optional TLS generation, Key Vault secret creation, module orchestration, and outputs

### Additional Thoughts

### 🔐 SSL and Kubernetes Integration (SSE-Only)
- **Edge Component (140-azureml)**: Owns TLS materials generation (optional) and Key Vault secret storage (cert/key)
- **IoT Ops (110-iot-ops)**: Owns Kubernetes manifests via apply scripts (azureml namespace, ServiceAccount, SecretProviderClass, SecretSync)
- **Extension Integration**: Uses `sslSecret` and `sslCname` configuration settings for TLS-enabled scenarios

### 🛡️ Private VNet Requirements
- **Cloud Workspace**: `public_network_access_enabled = false` enforced for edge scenarios
- **Network Isolation**: Support for `managed_network.isolation_mode = "AllowOnlyApprovedOutbound"`
- **Private Endpoints**: DNS zones `privatelink.api.azureml.ms` and `privatelink.notebooks.azure.net` for commercial cloud
