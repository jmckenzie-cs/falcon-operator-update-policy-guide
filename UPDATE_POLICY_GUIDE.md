# Basic Guide: Falcon Operator with Update Policy

This guide shows how to deploy the CrowdStrike Falcon Operator and Node Sensor using an update policy from your Falcon console for controlled sensor version management.

## Overview

Update policies allow you to control exactly which sensor version gets deployed through policies managed in your CrowdStrike Falcon console, rather than automatically using the latest available version.

## Prerequisites

1. **Kubernetes cluster** with admin access
2. **CrowdStrike Falcon API credentials** with the following scopes:
   - **Sensor Download** - Required for downloading sensor images
   - **Sensor update policies (read)** - Required for querying update policies
   - **Host group** (optional) - For advanced host management
3. **Access to CrowdStrike Falcon console** to create update policies

## Step 1: Create Update Policy in Falcon Console

Before deploying the sensor, create the update policy that will control which sensor version gets deployed:

### Creating Update Policies

1. **Login to your CrowdStrike Falcon console**
2. **Navigate to:** Host Management → Sensor Updates → Update Policies
3. **Create New Policy:**
   - **Name:** Choose a descriptive name (e.g., "k8s-production", "k8s-staging", "k8s-latest")
   - **Description:** Optional description of the policy purpose
   - **Sensor Version:** Select specific sensor version (e.g., "7.14", "7.15")
   - **Architecture:** Select architectures you need:
     - ✅ x86_64 (Intel/AMD)
     - ✅ ARM64 (if you have ARM nodes)
   - **Status:** ✅ Enable the policy
4. **Save** the policy
5. **Note the exact policy name** - you'll need this for the Kubernetes deployment

### Example Policy Configurations

**Production Policy:**
- Name: `k8s-production`
- Version: `7.14` (stable, well-tested version)
- Enabled: ✅

**Staging Policy:**
- Name: `k8s-staging`
- Version: `7.15` (newer version for testing)
- Enabled: ✅

**Latest Policy:**
- Name: `k8s-latest`
- Version: `7.16` (latest available)
- Enabled: ✅

## Step 2: Deploy the Falcon Operator

```bash
# Create operator namespace
kubectl create namespace falcon-operator

# Deploy the operator
kubectl apply -f https://github.com/CrowdStrike/falcon-operator/releases/latest/download/falcon-operator.yaml

# Verify deployment
kubectl get pods -n falcon-operator
```

Wait for the operator pod to be `Running`.

## Step 3: Create API Credentials Secret

```bash
# Create falcon-system namespace
kubectl create namespace falcon-system

# Create the secret (replace with your actual credentials)
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: falcon-api-secret
  namespace: falcon-system
type: Opaque
stringData:
  falcon-client-id: "YOUR_CLIENT_ID_HERE"
  falcon-client-secret: "YOUR_CLIENT_SECRET_HERE"
EOF
```

## Step 4: Deploy Node Sensor with Update Policy

```bash
# Deploy node sensor with update policy (replace "your-policy-name" with actual policy name)
cat <<EOF | kubectl apply -f -
apiVersion: falcon.crowdstrike.com/v1alpha1
kind: FalconNodeSensor
metadata:
  name: falcon-node-sensor
  namespace: falcon-system
spec:
  falconSecret:
    enabled: true
    namespace: falcon-system
    secretName: falcon-api-secret
  node:
    advanced:
      updatePolicy: "YOUR_POLICY_NAME"  # Reference your Falcon console policy
    backend: bpf
    tolerations:
    - effect: NoSchedule
      key: node-role.kubernetes.io/control-plane
      operator: Exists
    - effect: NoSchedule
      key: node-role.kubernetes.io/master
      operator: Exists
  falcon:
    tags:
    - "update-policy-managed"
EOF
```

## Step 5: Verify Deployment

```bash
# Check FalconNodeSensor status
kubectl get falconnodesensor -n falcon-system

# Check DaemonSet
kubectl get daemonset -n falcon-system

# Check pods are running
kubectl get pods -n falcon-system -o wide

# Check for any issues
kubectl describe falconnodesensor falcon-node-sensor -n falcon-system
```

## Monitoring and Verification

### Check Current Policy and Version
```bash
# Verify update policy is being used
kubectl get falconnodesensor falcon-node-sensor -n falcon-system -o jsonpath='{.spec.node.advanced.updatePolicy}'

# Check deployed sensor version
kubectl get daemonset -n falcon-system -o wide

# Monitor sensor status
kubectl describe falconnodesensor falcon-node-sensor -n falcon-system
```

### View Events
```bash
# Check for policy-related events
kubectl get events -n falcon-system --sort-by=.firstTimestamp

# Check operator logs
kubectl logs -n falcon-operator deployment/falcon-operator-controller-manager
```

## Troubleshooting

### Policy Not Found
If you see errors about policy not found:

```bash
# Check operator logs for API errors
kubectl logs -n falcon-operator deployment/falcon-operator-controller-manager | grep -i policy

# Verify API credentials
kubectl get secret falcon-api-secret -n falcon-system -o yaml
```

**Solutions:**
1. Verify policy name matches exactly (case-sensitive)
2. Ensure policy is enabled in Falcon console
3. Check API credentials have proper permissions

### Version Mismatch
If sensor doesn't deploy expected version:

1. Check policy version in Falcon console
2. Verify policy supports your architecture (x86_64/ARM64)
3. Ensure policy is enabled

## Update Policy vs AutoUpdate

| Feature | Update Policy | AutoUpdate |
|---------|--------------|------------|
| **Version Control** | Specific versions from console policies | Latest available from registry |
| **Change Management** | Controlled through Falcon console | Automatic based on registry |
| **Use Case** | Production, staged rollouts | Development, always-current |
| **Priority** | Higher (overrides autoUpdate if both set) | Lower (fallback option) |

## Key Benefits

✅ **Controlled Updates:** Use specific, tested sensor versions
✅ **Centralized Management:** Policies managed in Falcon console
✅ **Staged Rollouts:** Different policies for different environments
✅ **Compliance:** Predictable versions for change management
✅ **Multi-Architecture:** Support for x86_64 and ARM64

## Cleanup

```bash
# Remove node sensor
kubectl delete falconnodesensor falcon-node-sensor -n falcon-system

# Remove secret and namespace
kubectl delete secret falcon-api-secret -n falcon-system
kubectl delete namespace falcon-system

# Remove operator (optional)
kubectl delete -f https://github.com/CrowdStrike/falcon-operator/releases/latest/download/falcon-operator.yaml
kubectl delete namespace falcon-operator
```

## Summary

This approach gives you precise control over sensor versions through policies managed in your CrowdStrike Falcon console. The operator will query your API to get the version specified in the named policy and deploy exactly that version, providing predictable and controlled sensor management.