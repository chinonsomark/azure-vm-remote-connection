# Azure VM Remote Connection Lab

Hands-on lab demonstrating secure remote access to Azure virtual machines via RDP (Windows) and SSH (Linux), including Network Security Group (NSG) configuration, public IP identification, IP-restricted firewall rules, and browser-based access through Azure Bastion.

## Overview

This project covers the full workflow of provisioning Linux and Windows Azure VMs, securing them with NSG rules, connecting remotely via both traditional clients (SSH/RDP) and Azure Bastion, verifying access, and properly tearing down sessions and resources — core skills for cloud and DevOps system administration.

## Environment

### Linux VM
- **VM Name:** Linux-VM-Assessment
- **OS / Image:** Ubuntu Server 24.04 LTS
- **Region:** West Europe
- **VM Size:** Standard B2s v2 (2 vcpus, 8 GiB memory)
- **Public IP:** 20.54.142.190
- **Authentication Method:** Username and password (azureuser)
- **Virtual Network:** Linux-VM-Assessment-vnet

### Windows VM
- **VM Name:** Windows-VM-Assessment
- **OS / Image:** Windows Server 2022 Datacenter: Azure Edition
- **Region:** East US (Zone 1)
- **VM Size:** Standard D2ds v7
- **Public IP:** 20.115.52.141
- **Authentication Method:** Administrator username and password
- **Virtual Network:** Windows-VM-Assessment-vnet

## Task 1: NSG Configuration

| VM | Rule Name | Port | Protocol | Source | Action |
|---|---|---|---|---|---|
| Linux | Allow-SSH | 22 | TCP | Initially Any → restricted to 102.90.123.180/32 | Allow |
| Windows | RDP (auto-created) | 3389 | TCP | Any | Allow |

See: `screenshots/nsg-rule.png`, `screenshots/rdp-nsg-rule.png`, `screenshots/nsg-restricted.png`

## Task 2: Public IP Identification

- Linux VM public IP located on the Overview page: `20.54.142.190`
- Windows VM public IP located on the Overview page: `20.115.52.141`

See: `screenshots/public-ip.png`, `screenshots/windows-vm-overview.png`

## Tasks 3 & 4: Remote Connection

### Windows (RDP)
Connected using the Remote Desktop Connection client (`mstsc`) on Windows, authenticating with the administrator username and password set during VM creation.

See: `screenshots/rdp-connected.png`

### Linux (SSH)
Connected using:
```
ssh azureuser@20.54.142.190
```
Authenticated with username/password.

See: `screenshots/ssh-connected.png`

## Task 5: Verifying Access

Commands run inside the Linux VM to confirm successful access:
```
pwd
ls -la
df -h
cat /etc/os-release
```
On the Windows VM, access was verified via the Server Manager dashboard, confirming the OS, roles, and local server status.

See: `screenshots/verify-access.png`, `screenshots/rdp-connected.png`

## Task 6: Restricting NSG to a Specific Source IP

Updated the Linux VM's SSH inbound rule, changing **Source** from `Any` to my local public IP (`102.90.123.180/32`), then re-verified the connection still succeeded from that same machine — confirming the restriction correctly scoped access without locking out the legitimate client.

See: `screenshots/nsg-restricted.png`

## Azure Bastion Deployment

To demonstrate secure, browser-based access without exposing either VM's public IP for the connection itself, Azure Bastion was deployed separately for each VM's virtual network (the two VMs sit in different regions/VNets, so each required its own Bastion host and dedicated `AzureBastionSubnet`).

| VM | Bastion Name | Tier | Subnet |
|---|---|---|---|
| Linux-VM-Assessment | Linux-VM-Assessment-vnet-bastion | Basic | AzureBastionSubnet (auto-created) |
| Windows-VM-Assessment | Windows-VM-Assessment-vnet-bastion | Standard | AzureBastionSubnet (auto-created) |

Deployment steps for each:
1. Opened the VM's **Bastion** blade (Connect → Bastion).
2. Let Azure auto-create the required `AzureBastionSubnet` within the existing VNet.
3. Created a new public IP for the Bastion host itself (not the VM).
4. Deployed and waited ~5–10 minutes for provisioning to complete.

See: `screenshots/bastion-subnet.png`, `screenshots/bastion-deployment.png`, `screenshots/bastion-subnet-windows.png`, `screenshots/bastion-deployment-windows.png`

## Bastion Access Verification

Connected to both VMs directly through the browser using Bastion, with no SSH client, RDP client, or VM-facing public IP required for the session itself:

- **Linux:** Authenticated with VM Password (azureuser), opened a browser-based SSH terminal session.
- **Windows:** Authenticated with VM Password (administrator account), opened a full browser-based RDP desktop session.

See: `screenshots/bastion-linux-connect.png`, `screenshots/bastion-windows-connect.png`

## Task 7: Terminating Sessions and Cleanup

- Closed the SSH session cleanly using `exit`.
- Signed out of the RDP/Bastion Windows session through the Start menu rather than just closing the tab.
- Stopped (deallocated) the Linux VM via the Azure Portal to release compute resources and avoid unnecessary billing.
- Deleted both Azure Bastion resources after capturing verification screenshots, since Bastion bills hourly while deployed and is not needed for ongoing access.

See: `screenshots/vm-stopped.png`

## Troubleshooting Notes

- **SSH "Connection closed by host" right after entering password:** the Ubuntu image's `sshd_config` had `PasswordAuthentication` disabled by default. Resolved using Azure's **Run Command** feature to run `sed` against `/etc/ssh/sshd_config` and restart the `ssh` service, without needing prior SSH access.
- **VM size unavailable error during Windows VM creation:** `Standard_B1s` and `Standard_D2s_v3` both returned "insufficient quota" errors for the subscription/region. Resolved by selecting an available size (`Standard_D2ds_v7`) from the size picker instead.
- **Forgot VM username/password:** recovered the username via the auto-filled SSH command on the Connect blade, and reset the password using the **Reset password or keys** option on the same page.
- **Copy-pasting placeholder SSH command literally:** initially ran `ssh -i <private-key-file-path> azureuser@...` verbatim, which PowerShell misread as a redirection operator. Fixed by removing the placeholder entirely since password (not key) authentication was used.

## Summary

This lab reinforced how Azure VMs are accessed and secured in real-world cloud environments — from network-level firewall rules (NSGs) and public IP-based RDP/SSH access, down to fully private, browser-based connectivity through Azure Bastion, plus the operational discipline of clean session termination and resource cleanup to control cost.
