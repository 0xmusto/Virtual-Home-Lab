# Virtual Home Lab — Windows Server & Active Directory

A virtual home lab simulating a small-scale enterprise network environment. Built to develop practical, hands-on experience with Windows Server administration, Active Directory, and enterprise network design.

---

## Environment

| VM | OS | Role |
|---|---|---|
| DC01 | Windows Server 2022 Core | Domain Controller + DNS Server |
| CLIENT01 | Windows 10 Pro | Client Workstation |
| LINUX01 | Ubuntu Server | Linux Service Server |

All three virtual machines are configured and managed through Oracle VirtualBox.

![VirtualBox VM Overview](Screenshot_2026-05-16_102054.png)

**Network:** Each VM uses a NAT adapter for internet access and a Host-Only adapter for internal communication, mirroring how enterprise environments separate internal and external traffic.

**Static IPs:**

| VM | IP Address |
|---|---|
| DC01 | 192.168.56.10 |
| CLIENT01 | 192.168.56.20 |
| LINUX01 | 192.168.56.30 |

---

## Network Configuration

### DC01 — Static IP via Server Core

On DC01 (Windows Server Core), the static IP was configured through the built-in `sconfig` network menu. DHCP was disabled and the server was set to use itself as the preferred DNS server.

![DC01 Network Adapter Settings](Screenshot_2026-05-16_103406.png)

### LINUX01 — Static IP via Netplan

On the Ubuntu server, the static IP was configured by editing the Netplan configuration file at `/etc/netplan/01-netcfg.yaml`. DNS was pointed at DC01 (`192.168.56.10`) so the Linux server resolves domain names through the lab's own DNS server.

![Netplan Configuration](Screenshot_2026-05-16_103648.png)

After saving the file, the configuration was applied with:

```bash
sudo netplan apply
```

The result was verified using:

```bash
ip a
```

![LINUX01 IP Verification](Screenshot_2026-05-16_103601.png)

### CLIENT01 — Static IP via Windows GUI

On CLIENT01, the static IP was set through the TCP/IPv4 adapter properties. DNS was pointed at `192.168.56.10` (DC01) so the workstation can locate the domain controller for authentication.

![CLIENT01 TCP/IPv4 Settings](Screenshot_2026-05-16_103752.png)

---

## Active Directory

AD DS was installed and configured on DC01 via PowerShell. The domain controller runs Windows Server Core — no GUI, everything through the command line.

```powershell
# Install the AD DS role
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools

# Promote the server to Domain Controller and create the forest
Install-ADDSForest -DomainName "company.local"
```

**Domain:** `company.local`

DC01 runs Windows Server Core with no GUI. All server configuration is done through `sconfig` and PowerShell. The screenshot below shows the `sconfig` menu confirming the server is joined to `company.local` as `DC01`.

![DC01 SConfig Menu](Screenshot_2026-05-16_103107.png)

DC01 also serves as the DNS server for the environment. DNS is the backbone of Active Directory — clients rely on it to locate domain controllers and authentication services.

---

## Domain Structure

Active Directory was managed using `dsa.msc` (Active Directory Users and Computers).

**Organizational Units:**
- HR
- IT

**Security Groups:**
- `HR_Group`
- `IT_Group`

Users are assigned to groups rather than having permissions applied directly to individual accounts, which keeps access control scalable and maintainable.

---

## File Sharing & Permissions

A shared folder was created on DC01 to simulate company file access with role-based permissions.

```powershell
# Create the shared folder
New-Item -Path "c:\Shared" -ItemType Directory
```

![Creating the Shared Folder](Screenshot_2026-05-16_104136.png)

| Group | Permission |
|---|---|
| IT_Group | Full Control |
| HR_Group | Read-Only |

Permissions were verified by logging in as users from each group and testing read, write, and delete access.

---

## Group Policy

Group Policy Objects were created and linked to each OU to enforce department-level restrictions and centralized settings.

### HR_User_Restrictions GPO

A GPO named `HR_User_Restrictions` was created and linked to the HR OU. Security filtering was scoped specifically to `HR_Group` so the policy only applies to HR users.

![GPO Linked to HR OU](Screenshot_2026-05-14_170957.png)

### Control Panel Restriction

Inside the GPO, the **Prohibit access to Control Panel and PC settings** policy was enabled under:

`User Configuration > Policies > Administrative Templates > Control Panel`

![Group Policy Management Editor](Screenshot_2026-05-14_165044.png)

![Control Panel Restriction Enabled](Screenshot_2026-05-14_165106.png)

This prevents HR users from accessing Control Panel or the Settings app entirely — simulating a standard enterprise workstation lockdown.

```powershell
# Force an immediate Group Policy update on a machine
gpupdate /force

# View currently applied policies
gpresult /r

# Generate a full HTML Group Policy report
gpresult /h report.html
```

---

## Project Status

| Component | Status |
|---|---|
| Active Directory | Complete |
| DNS | Complete |
| Domain Environment | Complete |
| User & Group Management | Complete |
| Shared Folder & Permissions | Complete |
| Group Policy | Complete |
| Enterprise Feature Expansion | In Progress |

---

## Planned Additions

- Folder Redirection and Drive Mapping via GPO
- DHCP Server
- WSUS (Windows Server Update Services)
- VPN Configuration
- Linux Service Integration
- Backup and Recovery Testing
- Monitoring and Logging

---

## Technologies

Oracle VM VirtualBox, Windows Server 2022 Core, Windows 10 Pro, Ubuntu Server, PowerShell, Active Directory, DNS, Group Policy Management, NTFS Permissions
