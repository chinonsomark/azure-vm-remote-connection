# Azure VM Remote Connection Lab

Hands-on lab demonstrating secure remote access to Azure virtual machines via RDP (Windows) and/or SSH (Linux), including Network Security Group (NSG) configuration, public IP identification, and IP-restricted firewall rules.

## Overview

This project covers the full workflow of provisioning an Azure VM, securing it with NSG rules, connecting to it remotely, verifying access, and properly tearing down the session — core skills for cloud and DevOps system administration.

## Environment

- **VM Name:** _e.g. linux-ssh-vm_
- **OS / Image:** _e.g. Ubuntu Server 22.04 LTS_
- **Region:** _e.g. East US_
- **VM Size:** _e.g. Standard B1s_
- **Authentication Method:** _e.g. SSH key pair / password_

## Task 1: NSG Configuration

| Rule Name | Port | Protocol | Source | Action |
|---|---|---|---|---|
| Allow-SSH | 22 | TCP | _your IP/32_ | Allow |

_(Replace with your actual rule(s) — include both initial "Any" source and the restricted version from Task 6 if you want to show the before/after.)_

See: `screenshots/nsg-rule.png`

## Task 2: Public IP

The VM's public IP address was located on the Azure Portal VM Overview page: `<insert IP here>`

See: `screenshots/public-ip.png`

## Tasks 3 & 4: Remote Connection

### Windows (RDP)
- Connected using Remote Desktop Connection client (`mstsc`) / Microsoft Remote Desktop on Mac.
- Authenticated with the admin username/password set during VM creation.

See: `screenshots/rdp-connected.png`

### Linux (SSH)
- Connected using: `ssh -i <keyfile> azureuser@<public-ip>`
- Authenticated via SSH key pair / password.

See: `screenshots/ssh-connected.png`

## Task 5: Verifying Access

Commands run inside the VM to confirm successful access:

```
pwd
ls -la
df -h
cat /etc/os-release
```

See: `screenshots/verify-access.png`

## Task 6: Restricting NSG to a Specific Source IP

Updated the inbound rule's **Source** field from `Any` to my local public IP (`<your IP>/32`) to prevent unauthorized access from other addresses, then re-verified the connection still worked from my machine.

See: `screenshots/nsg-restricted.png`

## Task 7: Terminating the Session

- Logged out of the RDP/SSH session cleanly (`exit` / sign-out, not just closing the window).
- Stopped (deallocated) the VM from the Azure Portal afterward to avoid unnecessary billing and free up resources.

See: `screenshots/vm-stopped.png`

## Troubleshooting Notes

_Document any issues you ran into here, e.g.:_
- "Initial SSH connection timed out — NSG didn't yet have port 22 open. Resolved by adding the inbound rule in Step 3."
- "RDP showed a certificate warning — expected behavior for a self-signed cert, proceeded after confirming the IP matched."

## Summary

This lab reinforced how Azure VMs are accessed and secured in real-world cloud environments — from network-level firewall rules (NSGs) down to session-level access control and proper resource cleanup.
