# Forests Node (WSL2) Configuration

The `forests` node runs on WSL2 and has special networking requirements.

## CNI Configuration

Cilium does not fully support WSL2 due to kernel limitations. The forests node uses Bridge CNI instead:

- **k3s-one node**: Uses Cilium CNI (managed by Flux)
- **forests node**: Uses Bridge CNI (manually configured on the node)

## Setup Steps

### 1. CNI Configuration (Already Done)
The forests node has Bridge CNI configured at `/etc/cni/net.d/10-bridge.conflist`

### 2. Remove Cilium Taint (Manual Step Required)
After the node starts, remove the Cilium taint to allow pods to schedule:

```bash
kubectl taint nodes forests node.cilium.io/agent-not-ready:NoSchedule-
```

This taint is automatically applied by Cilium but needs to be manually removed since Cilium won't run on this node.

### 3. Verify
Check that pods can schedule and run:

```bash
kubectl get pods --all-namespaces --field-selector spec.nodeName=forests
```

## Why This Setup?

- Cilium requires eBPF features that are limited in WSL2
- See: https://github.com/cilium/cilium/issues/21542
- The forests node affinity rules in `helmvalues.yaml` prevent Cilium from scheduling there
- Bridge CNI works reliably on WSL2 without kernel modifications

## Excluded Services

The following services are excluded from running on the forests node:

- **node-feature-discovery**: Has issues with limited hardware access in WSL2 (crashes)
- **Cilium**: Not compatible with WSL2 kernel limitations

## Maintenance

The Bridge CNI configuration must persist across node restarts. It's stored in `/etc/cni/net.d/` within the WSL2 instance.
