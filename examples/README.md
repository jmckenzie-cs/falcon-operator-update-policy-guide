# Example Configurations

This directory contains example YAML files for different deployment scenarios.

## Files

- **[secret.yaml](secret.yaml)** - Kubernetes secret with correct field names for `falconSecret`
- **[falconnodesensor.yaml](falconnodesensor.yaml)** - Basic FalconNodeSensor configuration
- **[production-falconnodesensor.yaml](production-falconnodesensor.yaml)** - Production environment with resource limits
- **[staging-falconnodesensor.yaml](staging-falconnodesensor.yaml)** - Staging environment for testing

## Usage

1. **Replace placeholders** in the files with your actual values:
   - `YOUR_CLIENT_ID_HERE` → Your CrowdStrike API Client ID
   - `YOUR_CLIENT_SECRET_HERE` → Your CrowdStrike API Client Secret
   - `YOUR_POLICY_NAME` → Your update policy name from Falcon console

2. **Apply the configurations**:
   ```bash
   # Create secret
   kubectl apply -f secret.yaml

   # Deploy node sensor (choose one)
   kubectl apply -f falconnodesensor.yaml
   # OR for production
   kubectl apply -f production-falconnodesensor.yaml
   # OR for staging
   kubectl apply -f staging-falconnodesensor.yaml
   ```

## Key Configuration Notes

### Secret Field Names
- ✅ Use `falcon-client-id` and `falcon-client-secret`
- ❌ NOT `client_id` and `client_secret` (old format)

### FalconNodeSensor Structure
- ✅ Use `falconSecret: {enabled: true, namespace: ..., secretName: ...}`
- ❌ NOT `falcon_secret: {name: ...}` (old format)

### CID Auto-Discovery
- The operator automatically discovers your CID from API credentials
- No need to manually specify `falcon-cid` in the secret