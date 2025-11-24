# VCF Edge - Single-Node vSphere Supervisor with ArgoCD Automation Script

This PowerShell automation script streamlines the deployment of a single-node vSphere Supervisor in VMware Cloud Foundation (VCF) 9.x environments based on [VMware's design guidance](https://blogs.vmware.com/cloud-foundation/2025/07/14/modernizing-your-edge-with-single-node-vsphere-supervisor-in-vmware-cloud-foundation-9-0/).

## Overview

`OneNodeDeployment.ps1` provides end-to-end automation for deploying a single-node vSphere Supervisor, handling everything from initial infrastructure setup to ArgoCD installation for GitOps workflows.

### Key Features

- **Infrastructure Setup**: ESX cluster creation, datastore configuration, and storage policy setup
- **Network Configuration**: Virtual Distributed Switch (VDS) setup with automated port group configuration
- **Supervisor Deployment**: Complete vSphere Supervisor cluster deployment with Foundation Load Balancer
- **ArgoCD Integration**: Automated installation and configuration of ArgoCD for GitOps workflows
- **Comprehensive Validation**: Deep validation of configuration files with detailed error reporting
- **Robust Error Handling**: Structured error codes and detailed logging for troubleshooting

## Prerequisites

### Software Requirements

- **PowerShell**: Version 7.0 or later
- **VCF.PowerCLI**: Version 9.0 or later
- **Network Connectivity**: Access to vCenter Server and ESXi host(s)
- **Credentials**: Valid vCenter administrator credentials

### Required Files

1. **Configuration Files** (JSON):
   - `infrastructure.json` - vCenter, cluster, network, and storage settings
   - `supervisor.json` - Supervisor Cluster and VKS configuration

2. **ArgoCD Files** (YAML):
   - `1.0.1-24896502.yml` - ArgoCD operator manifest
   - `argocd-deployment.yml` - ArgoCD deployment configuration

## Configuration

### Infrastructure Configuration (`infrastructure.json`)

This file defines your infrastructure components:

```json
{
  "common": {
    "vCenterName": "vc01.example.com",
    "vCenterUser": "administrator@vsphere.local",
    "esxHost": "esx01.example.com",
    "esxUser": "root",
    "datacenterName": "dc01",
    "clusterName": "cl02",
    "supervisorName": "supervisor-01",
    "datastore": {
      "datastoreName": "datastore-01"
    },
    "storagePolicy": {
      "storagePolicyName": "VMFS-Storage-Policy",
      "storagePolicyType": "VMFS"
    },
    "virtualDistributedSwitch": {
      "vdsName": "Prod-VDS2",
      "vdsVersion": "9.0.0",
      "numUplinks": 2,
      "nicList": [{"name": "vmnic1"}],
      "portGroups": [
        {"name": "tkgmanagement", "vlanId": "1000"},
        {"name": "flbmgmtnetwork", "vlanId": "1001"},
        {"name": "workloadnetwork", "vlanId": "1002"},
        {"name": "virtualsrvnetwork", "vlanId": "1003"}
      ]
    },
    "argoCD": {
      "argoCdOperatorYamlPath": "C:\\path\\to\\1.0.1-24896502.yml",
      "argoCdDeploymentYamlPath": "C:\\path\\to\\argocd-deployment.yml",
      "contextName": "vcf-context-01",
      "nameSpace": "argocd",
      "vmClass": ["best-effort-2xlarge", "best-effort-4xlarge"]
    }
  }
}
```

### Supervisor Configuration (`supervisor.json`)

This file defines your Supervisor Cluster settings:

```json
{
  "supervisorSpec": {
    "controlPlaneVMCount": 1,
    "controlPlaneSize": "TINY"
  },
  "tkgsComponentSpec": {
    "foundationLoadBalancerComponents": {
      "flbName": "flb-sn1",
      "flbSize": "SMALL",
      "flbAvailability": "SINGLE_NODE",
      "flbVipStartIP": "10.11.20.201",
      "flbVipIPCount": 50
    },
    "tkgsMgmtNetworkSpec": {
      "tkgsMgmtNetworkName": "tkgmanagement",
      "tkgsMgmtNetworkGatewayCidr": "10.11.10.1/24",
      "tkgsMgmtNetworkStartingIp": "10.11.10.100",
      "tkgsMgmtNetworkIPCount": 7
    },
    "tkgsPrimaryWorkloadNetwork": {
      "tkgsPrimaryWorkloadNetworkName": "workloadnetwork",
      "tkgsPrimaryWorkloadNetworkGatewayCidr": "10.11.16.1/24",
      "tkgsPrimaryWorkloadNetworkStartingIp": "10.11.16.101",
      "tkgsPrimaryWorkloadNetworkIPCount": 100
    }
  }
}
```

## Usage

### Basic Usage

Run with default configuration files:

```powershell
.\OneNodeDeployment.ps1
```

### Advanced Usage

**Custom configuration files:**
```powershell
.\OneNodeDeployment.ps1 -infrastructureJson "config/site-a-infrastructure.json" -supervisorJson "config/site-a-supervisor.json"
```

**Debug mode for troubleshooting:**
```powershell
.\OneNodeDeployment.ps1 -logLevel DEBUG
```

**Check script version:**
```powershell
.\OneNodeDeployment.ps1 -version
```

**Minimal console output:**
```powershell
.\OneNodeDeployment.ps1 -logLevel WARNING
```

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `infrastructureJson` | String | `"infrastructure.json"` | Path to infrastructure configuration file |
| `supervisorJson` | String | `"supervisor.json"` | Path to Supervisor configuration file |
| `logLevel` | String | `"INFO"` | Console log level: `DEBUG`, `INFO`, `ADVISORY`, `WARNING`, `EXCEPTION`, `ERROR` |
| `version` | Switch | - | Display script version and exit |

## Deployment Process

The script performs the following steps:

1. **Validation**: Validates JSON configuration files for syntax and required properties
2. **Connection**: Establishes connections to vCenter and ESXi host
3. **Cluster Setup**: Creates ESXi cluster and adds host
4. **Storage Configuration**: Creates datastore and configures storage policies
5. **Network Setup**: Creates Virtual Distributed Switch and port groups
6. **Supervisor Deployment**: Deploys vSphere Supervisor with Foundation Load Balancer
7. **ArgoCD Installation**: Installs and configures ArgoCD operator and services

## Logging

Logs are automatically created in the `logs/` directory:

- **Format**: `logs/OneNodeDeployment-YYYY-MM-DD.log`
- **Behavior**: All log levels are written to file regardless of console `logLevel` setting
- **Console Output**: Filtered based on `logLevel` parameter

### Log Levels (Lowest to Highest)

- `DEBUG`: Detailed diagnostic information
- `INFO`: General progress messages
- `ADVISORY`: Important notices
- `WARNING`: Potential issues
- `EXCEPTION`: Caught exceptions
- `ERROR`: Failures requiring attention

## Error Handling

The script uses structured error codes for easy troubleshooting:

| Error Category | Error Codes | Description |
|----------------|-------------|-------------|
| Connection | `ERR_NOT_CONNECTED`, `ERR_TIMEOUT` | Connection failures |
| Network | `ERR_VDS_*`, `ERR_PORTGROUP_*`, `ERR_NIC_CONFIG` | Network configuration issues |
| Version | `ERR_VERSION_*` | Version compatibility problems |
| Kubernetes | `ERR_KUBECTL_*` | kubectl command failures |
| ArgoCD | `ERR_ARGOCD_*` | ArgoCD deployment issues |
| Validation | `ERR_VALIDATION` | Configuration validation failures |

## Troubleshooting

1. **Check Logs**: Review the dated log file in the `logs/` directory
2. **Run with DEBUG**: Execute with `-logLevel DEBUG` for verbose output
3. **Validate JSON**: Ensure JSON configuration files are properly formatted
4. **Verify Prerequisites**: Confirm PowerShell version, PowerCLI modules, and network connectivity
5. **Review Error Codes**: Use error codes in logs to identify specific failure points

## Documentation

- [vSphere Supervisor Documentation](https://techdocs.broadcom.com/us/en/vmware-cis/vcf/vcf-9-0-and-later/9-0/vsphere-supervisor-installation-and-configuration/vsphere-supervisor-concepts/vsphere-iaas-control-plane-concepts/what-is-vsphere-with-tanzu.html)
- [VMware Cloud Foundation Documentation](https://docs.vmware.com/en/VMware-Cloud-Foundation/index.html)
- [Design Guidance Blog Post](https://blogs.vmware.com/cloud-foundation/2025/07/14/modernizing-your-edge-with-single-node-vsphere-supervisor-in-vmware-cloud-foundation-9-0/)

## Version

Current version: **1.0.0.2**

## License

Copyright (c) 2025 Broadcom. All Rights Reserved.
Copyright (c) CA, Inc. All rights reserved.

You are hereby granted a non-exclusive, worldwide, royalty-free license under CA, Inc.'s copyrights to use, copy, modify, and distribute this software in source code or binary form for use in connection with CA, Inc. products.

This copyright notice shall be included in all copies or substantial portions of the software.

**THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.**

## File Checksums

```
b2f2dc47f9e6569d2202e6371699d3152caee2b860454199ab02a8b011f6b2d4  1.0.1-24896502.yml
ab54160996d8f5f636dd7c91eb7bc5ccfe45323e49e67fe82c07ee21df9fe03a  Admin.Guide.Single.Node.Supervisor.rtf
0c214a60ebeff7ef8cff56374d3aee456c709a0d41ea3812de73ab88446e7ff9  argocd-deployment.yml
080ce167e3247763b785700414d0ce306e32a666d8861e4f3670e3349ffc99fe  infrastructure.json
01ba4719c80b6fe911b091a7c05124b64eeece964e09c058ef8f9805daca546b  LICENSE
4807c2962bc292fa746d1945158464d55dbd6baa08d1f7d9aa22cbd4cc24f96f  OneNodeDeployment.ps1
2cf4797d25729c1ca07d7f02bcff0c4ebdd5a5f25765fad0d944ca4c95861089  README.md
234adfd9ebdd48da4ec6f8c473f950eabfd1c16687a0df995fa91d368952d641  supervisor.json
```
