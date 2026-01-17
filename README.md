# Falcon Operator Update Policy Guide

A comprehensive guide for deploying CrowdStrike Falcon Operator with Update Policy management for controlled sensor version deployment in Kubernetes.

## Overview

This guide provides step-by-step instructions for deploying the CrowdStrike Falcon Operator with update policies from your Falcon console. Choose between two deployment options:

- **Option 1**: Node Sensor only (basic endpoint protection)
- **Option 2**: Complete Falcon platform (Node Sensor + Admission Controller + Image Assessment)

Update policies allow you to control exactly which sensor version gets deployed, rather than automatically using the latest available version.

## Key Features

- ✅ **Controlled Updates**: Use specific, tested sensor versions
- ✅ **Centralized Management**: Policies managed in Falcon console
- ✅ **Staged Rollouts**: Control sensor versions through update policies
- ✅ **Security**: Proper secret management with `falconSecret` functionality
- ✅ **Multi-Architecture**: Support for x86_64 and ARM64

## Quick Start

1. **Prerequisites**: Kubernetes cluster + CrowdStrike Falcon API credentials
2. **Create Update Policy**: In your Falcon console (Host Management → Sensor Updates → Update Policies)
3. **Deploy Operator**: `kubectl apply -f https://github.com/CrowdStrike/falcon-operator/releases/latest/download/falcon-operator.yaml`
4. **Create Secret**: With your API credentials using correct field names
5. **Choose Deployment**: Option 1 (Node Sensor) or Option 2 (Complete Platform)

## Required API Permissions

Your CrowdStrike API client needs these scopes:
- **Sensor Download** - Required for downloading sensor images
- **Falcon Images Download: Read** - Required for registry access
- **Sensor update policies (read)** - Required for querying update policies

## Documentation

- **[UPDATE_POLICY_GUIDE.md](UPDATE_POLICY_GUIDE.md)** - Complete step-by-step guide
- **[examples/](examples/)** - Example configurations

## Troubleshooting

**Common Issues:**
1. **Authentication errors** - Check secret field names (`falcon-client-id` vs `client_id`)
2. **Policy not found** - Ensure policy is enabled in Falcon console
3. **Permission denied** - Add "Falcon Images Download: Read" API scope
4. **CID issues** - Remove placeholder CID values, let operator auto-discover

## Contributing

This guide was created from real deployment experience and troubleshooting. Feel free to contribute improvements or report issues.

## License

This guide is provided for educational and operational purposes. CrowdStrike Falcon and its components are proprietary software.