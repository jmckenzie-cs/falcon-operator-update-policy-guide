# Example Configurations

This directory contains example YAML files for deploying Falcon with update policies.

## Deployment Options

### Option 1: Node Sensor Only
- **[secret.yaml](secret.yaml)** - Kubernetes secret with correct field names
- **[falconnodesensor.yaml](falconnodesensor.yaml)** - Node sensor only deployment

### Option 2: Complete Falcon Platform
- **[secret.yaml](secret.yaml)** - Kubernetes secret (same for both options)
- **[falcondeployment.yaml](falcondeployment.yaml)** - Complete platform deployment

## Components in Complete Platform (Option 2)

The FalconDeployment includes:
- ✅ **Falcon Sensor for Linux** - Node-level endpoint protection
- ✅ **Kubernetes Admission Controller** - Webhook-based admission control
- ✅ **Image Assessment at Runtime** - Container image vulnerability scanning

## Usage

1. **Replace placeholders** with your actual values:
   - `YOUR_CLIENT_ID_HERE` → Your CrowdStrike API Client ID
   - `YOUR_CLIENT_SECRET_HERE` → Your CrowdStrike API Client Secret
   - `YOUR_POLICY_NAME` → Your update policy name from Falcon console

2. **Choose and apply your deployment option**:

   **Option 1 - Node Sensor Only:**
   ```bash
   kubectl apply -f secret.yaml
   kubectl apply -f falconnodesensor.yaml
   ```

   **Option 2 - Complete Platform:**
   ```bash
   kubectl apply -f secret.yaml
   kubectl apply -f falcondeployment.yaml
   ```

## Key Configuration Notes

### Secret Field Names
- ✅ Use `falcon-client-id` and `falcon-client-secret`
- ❌ NOT `client_id` and `client_secret` (old format)

### UpdatePolicy Support
- Both deployment options support update policies
- Configure in Falcon console: Host Management → Sensor Updates → Update Policies
- Ensure policy is **enabled** in the console

### CID Auto-Discovery
- The operator automatically discovers your CID from API credentials
- No need to manually specify `falcon-cid` in the secret