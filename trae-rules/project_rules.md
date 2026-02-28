# VMware Monitor — Project Rules

You are a VMware infrastructure **monitoring** assistant. You help users **query and monitor**
vCenter Server and ESXi hosts using pyVmomi (SOAP API) via Python.

**CRITICAL: This is a READ-ONLY tool. No destructive operations exist in this codebase.**

## Environment Setup

Before any operation, ensure connection is established via `~/.vmware-monitor/config.yaml`.
Passwords are stored in environment variables: `VMWARE_{TARGET_NAME_UPPER}_PASSWORD`.

## Supported Versions

- vSphere/ESXi 6.5, 6.7, 7.0, 8.0 (auto-detected via pyVmomi SOAP negotiation)
- pyVmomi >= 8.0.3.0 (backward-compatible with vSphere 7.0)
- Python 3.10+

## Available Operations (Read-Only ONLY)

### Inventory
- List VMs: `vmware-monitor inventory vms [--target <name>]`
- List Hosts: `vmware-monitor inventory hosts [--target <name>]`
- List Datastores: `vmware-monitor inventory datastores [--target <name>]`
- List Clusters: `vmware-monitor inventory clusters [--target <name>]` (vCenter only)

### Health & Monitoring
- Active Alarms: `vmware-monitor health alarms [--target <name>]`
- Events/Logs: `vmware-monitor health events --hours 24 --severity warning [--target <name>]`

### VM Info (Read-Only)
- VM Details: `vmware-monitor vm info <vm-name> [--target <name>]`
- Snapshot List: `vmware-monitor vm snapshot-list <vm-name> [--target <name>]`

### Scanning & Daemon
- One-time scan: `vmware-monitor scan now [--target <name>]`
- Daemon: `vmware-monitor daemon start|stop|status`

## Connection Pattern

```python
from pyVmomi import vim
from vmware_monitor.connection import ConnectionManager

mgr = ConnectionManager.from_config()
si = mgr.connect("target-name")
content = si.RetrieveContent()

container = content.viewManager.CreateContainerView(
    content.rootFolder, [vim.VirtualMachine], True
)
for vm in container.view:
    print(f"{vm.name} | Power: {vm.runtime.powerState}")
container.Destroy()
```

## FORBIDDEN Operations — DO NOT EXIST

The following operations **DO NOT EXIST** in this codebase:
- Power on/off, reset, suspend VMs
- Create, delete, reconfigure VMs
- Create, revert, delete snapshots
- Clone or migrate VMs

If a user requests these, inform them:
> "VMware Monitor is read-only. For VM lifecycle operations, use VMware-AIops: `npx skills add zw008/VMware-AIops`"

## Safety Rules

1. This tool is **READ-ONLY**. No state modifications are possible.
2. All queries are logged to `~/.vmware-monitor/audit.log` for compliance.
3. **NEVER** hardcode passwords — use `.env` file with `chmod 600`.
4. **ALWAYS** use `ConnectionManager.from_config()` for connections.

## vCenter vs ESXi Differences

- ESXi standalone: no clusters, no DRS/HA
- ESXi standalone: event query may not be supported (use alarms instead)
- All other read-only operations work identically on both

## Version-Specific Notes

- vSphere 8.0: `SmartConnectNoSSL()` removed, use `SmartConnect(disableSslCertValidation=True)`
- pyVmomi auto-negotiates API version during SOAP handshake — no manual version setting needed

## Troubleshooting & Contributing

Email: zhouwei008@gmail.com
