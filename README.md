# Azure Arc - Windows Server Software Assurance & Azure Benefits Policy

This repository contains an Azure Policy template that automatically enables Windows Server license benefits when businesses onboard on-premises servers with Azure Arc.

## Overview

This policy automatically configures Arc-enabled Windows Servers to:
1. **Attest Software Assurance coverage** - Declares that the server is covered by Software Assurance
2. **Enable Azure Benefits** - Activates Azure Hybrid Benefit for the Arc-enabled server

When a new on-premises Windows Server is connected to Azure via Arc, this policy ensures proper license management is enabled.

## Repository Structure

```
.
├── README.md                           # This file
├── policy/
│   ├── azurepolicy.json               # Policy definition
│   └── azurepolicy.parameters.json    # Policy parameters
├── assignment/
│   ├── policy-assignment.json         # Assignment template
│   └── parameters.json                # Assignment parameters
└── docs/
    ├── deployment-guide.md            # Deployment instructions
    └── testing-guide.md               # Testing procedures
```

## Prerequisites

- Azure subscription with appropriate permissions
- Azure Arc-enabled servers infrastructure
- Azure Policy Contributor role or higher
- Azure PowerShell or Azure CLI installed

## Quick Start

### 1. Create Policy Definition

```powershell
# Using Azure PowerShell
New-AzPolicyDefinition `
    -Name "arc-windows-server-license-sa" `
    -DisplayName "Enable Windows Server Software Assurance for Arc Servers" `
    -Description "Automatically enables Windows Server SA and Azure Benefits" `
    -Policy "policy/azurepolicy.json" `
    -Parameter "policy/azurepolicy.parameters.json" `
    -Mode "Indexed"
```

```bash
# Using Azure CLI
az policy definition create \
    --name "arc-windows-server-license-sa" \
    --display-name "Enable Windows Server Software Assurance for Arc Servers" \
    --description "Automatically enables Windows Server SA and Azure Benefits" \
    --rules "policy/azurepolicy.json" \
    --params "policy/azurepolicy.parameters.json" \
    --mode "Indexed"
```

### 2. Assign Policy

```powershell
# Using Azure PowerShell
New-AzPolicyAssignment `
    -Name "arc-license-sa-assignment" `
    -DisplayName "Arc Windows License Software Assurance" `
    -Scope "/subscriptions/{subscription-id}" `
    -PolicyDefinition $(Get-AzPolicyDefinition -Name "arc-windows-server-license-sa") `
    -AssignIdentity `
    -Location "eastus"
```

```bash
# Using Azure CLI
az policy assignment create \
    --name "arc-license-sa-assignment" \
    --display-name "Arc Windows License Software Assurance" \
    --scope "/subscriptions/{subscription-id}" \
    --policy "arc-windows-server-license-sa" \
    --assign-identity \
    --location "eastus"
```

## Policy Details

### Target Resources
- **Resource Type**: `Microsoft.HybridCompute/machines`
- **OS Type**: Windows
- **Effect**: DeployIfNotExists

### What It Does

1. **Detects** newly onboarded or existing Arc-enabled Windows Servers
2. **Evaluates** compliance by checking if the license profile exists with:
   - `softwareAssuranceCustomer` set to `true`
   - `subscriptionStatus` set to `Enabled`
3. **Deploys** the license profile configuration if non-compliant:
   - Sets `softwareAssuranceCustomer: true` to attest Software Assurance coverage
   - Sets `subscriptionStatus: Enable` to activate Azure Benefits
4. **Ensures** compliance across all Arc-enabled Windows Servers

### License Types Supported

- Windows Server Datacenter
- Windows Server Standard

## Configuration Options

The policy supports the following parameters:

| Parameter | Description | Allowed Values | Default |
|-----------|-------------|----------------|----------|
| `effect` | Policy effect | DeployIfNotExists, AuditIfNotExists, Disabled | DeployIfNotExists |
| `licenseType` | Windows Server license type | Windows Server, WindowsServerDatacenter | WindowsServerDatacenter |

## Compliance and Monitoring

After deployment:
1. Navigate to Azure Policy in the Azure Portal
2. View compliance status for "Arc Windows License Software Assurance"
3. Review non-compliant resources
4. Trigger remediation tasks as needed

## Testing

See [docs/testing-guide.md](docs/testing-guide.md) for detailed testing procedures.

## Troubleshooting

### Common Issues

**Policy not triggering:**
- Verify the Arc agent is running on the server
- Check that the server OS is Windows
- Ensure the policy assignment has identity assigned

**License profile not applying:**
- Verify managed identity has required permissions (Azure Arc ScVmm VM Contributor role)
- Check Azure Hybrid Benefit eligibility
- Confirm Software Assurance coverage for the server
- Review activity logs for deployment errors

## Contributing

Contributions are welcome! Please feel free to submit issues and pull requests.

## Resources

- [Azure Arc Documentation](https://docs.microsoft.com/azure/azure-arc/)
- [Azure Policy Documentation](https://docs.microsoft.com/azure/governance/policy/)
- [Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-benefit/)
- [Software Assurance](https://www.microsoft.com/licensing/licensing-programs/software-assurance-default)

## License

MIT License - See LICENSE file for details
