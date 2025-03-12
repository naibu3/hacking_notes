# Tools

- The [[ActiveDirectory PowerShell module]] (MS signed and works even in PowerShell CLM)
https://learn.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2022-ps
https://github.com/samratashok/ADModule
```powershell
Import-Module C:\AD\Tools\ADModule-master\Microsoft.ActiveDirectory.Management.dll
Import-Module C:\AD\Tools\ADModule-master\ActiveDirectory\ActiveDirectory.psd1
```

- BloodHound (C# and PowerShell Collectors)
https://github.com/BloodHoundAD/BloodHound

- [[PowerView]] (PowerShell)
https://github.com/ZeroDayLab/PowerSploit/blob/master/Recon/PowerView.ps1
```powershell
. C:\AD\Tools\PowerView.ps1
```

- SharpView (C#) - Doesn't support filtering using Pipeline
https://github.com/tevora-threat/SharpView/

# Enumeration

## Get object domain

- Current:
```powershell
Get-Domain (PowerView)
Get-ADDomain (ActiveDirectory Module)
```

- Another domain:
```powershell
Get-Domain -Domain moneycorp.local (PowerView)
Get-ADDomain -Identity moneycorp.local (ActiveDirectory Module)
```

## Get object domain attributes

- Get domain SID (for the current domain):
```powershell
Get-DomainSID (PowerView)
(Get-ADDomain).DomainSID (ActiveDirectory Module)
```

- Get domain policy for the current domain
```powershell
Get-DomainPolicyData
(Get-DomainPolicyData).systemaccess
```

- Get domain policy for another domain
```powershell
(Get-DomainPolicyData -domain moneycorp.local).systemaccess
```

- Get domain controllers for the current domain:
```powershell
Get-DomainController (PowerView)
Get-ADDomainController (ActiveDirectory Module)
```

- Get domain controllers for another domain:
```powershell
Get-DomainController -Domain moneycorp.local
Get-ADDomainController -DomainName moneycorp.local -Discover
```

- Get *OUs* in a domain:
```powershell
Get-DomainOU
Get-ADOrganizationalUnit -Filter * -Properties *
```

## Get users

- Get a list of *users* in the current domain:
```powershell
Get-DomainUser
Get-DomainUser -Identity student1
```
> PowerView

```powershell
Get-ADUser -Filter * -Properties *
Get-ADUser -Identity student1 -Properties *
```
> ActiveDirectory Module

- Get list of all *properties* for users in the current domain:
```powershell
Get-DomainUser -Identity student1 -Properties *
Get-DomainUser -Properties samaccountname,logonCount
```
> PowerView

```powershell
Get-ADUser -Filter * -Properties * | select -First 1 | Get-Member -MemberType *Property | select Name
Get-ADUser -Filter * -Properties * | select name,logoncount,@{expression={[datetime]::fromFileTime($_.pwdlastset)}}
```
> ActiveDirectory Module

- Search for a particular string in a user's attributes:
```powershell
Get-DomainUser -LDAPFilter "Description=*built*" | Select name,Description
```
> PowerView

```powershell
Get-ADUser -Filter 'Description -like "*built*"' -Properties Description | select name,Description
```
> ActiveDirectory Module

- Get *actively logged users* on a computer (needs local admin rights on the target):
```powershell
Get-NetLoggedon -ComputerName dcorp-adminsrv
```

- Get locally logged users on a computer (needs remote registry on the target - started by-default on server OS):
```powershell
Get-LoggedonLocal -ComputerName dcorp-adminsrv
```

- Get the last logged user on a computer (needs administrative rights and remote registry on the target):
```powershell
Get-LastLoggedOn -ComputerName dcorp-adminsrv
```

### User Hunting

- Find all machines on the current domain where the current user has local admin access:
```powershell
Find-LocalAdminAccess -Verbose
```
> This function queries the DC of the current or provided domain for a list of computers (`Get-NetComputer`) and then use multi-threaded `Invoke-CheckLocalAdminAccess` on each machine.

This can also be done with the help of remote administration tools like WMI and PowerShell remoting. Pretty useful in cases ports (RPC and SMB) used by Find-LocalAdminAccess are blocked. See `Find-WMILocalAdminAccess.ps1` and `Find-PSRemotingLocalAdminAccess.ps1`.

- Find computers where a domain admin (or specified user/group) has sessions:
```powershell
Find-DomainUserLocation -Verbose
Find-DomainUserLocation -UserGroupIdentity "RDPUsers"
```
> This function queries the DC of the current or provided domain for members of the given group (Domain Admins by default) using `Get-DomainGroupMember`, gets a
list of computers (`Get-DomainComputer`) and list sessions and logged on users (`Get-NetSession/Get-NetLoggedon`) from each machine.

Note that for Server 2019 and onwards, local administrator privileges are required to list sessions.

- Find computers where a domain admin session is available and current user has admin access (`uses Test-AdminAccess`):
```powershell
Find-DomainUserLocation -CheckAccess
```

- Find computers (File Servers and Distributed File servers) where a domain admin session is available:
```powershell
Find-DomainUserLocation -Stealth
```

- List sessions on remote machines (https://github.com/Leo4j/Invoke-SessionHunter):
```powershell
Invoke-SessionHunter -FailSafe
```
> Above command doesn’t need admin access on remote machines. Uses Remote Registry and queries `HKEY_USERS` hive.

An opsec friendly command would be
```powershell
Invoke-SessionHunter -NoPortScan -Targets C:\AD\Tools\servers.txt
```

## Get computers

- Get a list of computers in the current domain:
```powershell
Get-DomainComputer | select Name
Get-DomainComputer -OperatingSystem "*Server 2022*"
Get-DomainComputer -Ping
```
> PowerView

```powershell
Get-ADComputer -Filter * | select Name
Get-ADComputer -Filter * -Properties *
Get-ADComputer -Filter 'OperatingSystem -like "*Server 2022*"' -Properties OperatingSystem | select Name,OperatingSystem
Get-ADComputer -Filter * -Properties DNSHostName | %{Test-Connection -Count 1 -ComputerName $_.DNSHostName}
```
> ActiveDirectory Module

## Get groups

- Get all the groups in the current domain:
```powershell
Get-DomainGroup | select Name
Get-DomainGroup -Domain <targetdomain>
```
> PowerView

```powershell
Get-ADGroup -Filter * | select Name
Get-ADGroup -Filter * -Properties *
```
> ActiveDirectory Module

- Get all groups containing the word "admin" in group name:
```powershell
Get-DomainGroup *admin* (PowerView)
Get-ADGroup -Filter 'Name -like "*admin*"' | select Name (ActiveDirectory Module)
```

- Get all the members of the Domain Admins group:
```powershell
Get-DomainGroupMember -Identity "Domain Admins" -Recurse (PowerView)
Get-ADGroupMember -Identity "Domain Admins" -Recursive (ActiveDirectory Module)
```

- Get the group membership for a user:
```powershell
Get-DomainGroup -UserName "student1" (PowerView)
Get-ADPrincipalGroupMembership -Identity student1 (ActiveDirectory Module)
```

- List all the local groups on a machine (needs administrator privs on non-dc machines) :
```powershell
Get-NetLocalGroup -ComputerName dcorp-dc
```

- Get members of the local group "Administrators" on a machine (needs administrator privs on non-dc machines) :
```powershell
Get-NetLocalGroupMember -ComputerName dcorp-dc -GroupName Administrators
```

## Get shares and files

- Find shares on hosts in current domain:
```powershell
Invoke-ShareFinder -Verbose
```

- Find sensitive files on computers in the domain:
```powershell
Invoke-FileFinder -Verbose
```

- Get all fileservers of the domain:
```powershell
Get-NetFileServer
```

## Group Policies (GPO)

Group Policy provides the ability to manage configuration and changes easily and centrally in AD. Allows configuration of -
- Security settings
-  Registry-based policy settings
- Group policy preferences like startup/shutdown/log-on/logoff scripts settings
- Software installation
- GPO can be abused for various attacks like privesc, backdoors, persistence etc.

- Get list of GPO in current domain:
```powershell
Get-DomainGPO
Get-DomainGPO -ComputerIdentity dcorp-student1
```

- Get GPO(s) which use Restricted Groups or groups.xml for interesting users:
```powershell
Get-DomainGPOLocalGroup
```

- Get users which are in a local group of a machine using GPO:
```powershell
Get-DomainGPOComputerLocalGroupMapping -ComputerIdentity dcorp-student1
```

- Get machines where the given user is member of a specific group:
```powershell
Get-DomainGPOUserLocalGroupMapping -Identity student1 -Verbose
```

- Get GPO applied on an OU. Read GPOname from gplink attribute from Get-NetOU:
```powershell
Get-DomainGPO -Identity "{0D1CC23D-1F20-4EEE-AF64-D99597AE2A6E}"
```

## Access Control Lists (ACL)

Enables control on the ability of a process to access objects and other resources in active directory based on:
- Access Tokens (security context of a process - identity and privs of user)
- Security Descriptors (SID of the owner, Discretionary ACL (DACL) and System ACL (SACL))

They are lists of *Access Control Entries* (ACE) - ACE corresponds to individual permission or audits access. Who has permission and what can be done on an object? There are two types:
- DACL - Defines the permissions trustees (a user or group) have on an object.
- SACL - Logs success and failure audit messages when an object is accessed.

ACLs are vital to security architecture of AD.

![[ACLs.png]]

- Get the ACLs associated with the specified object:
```powershell
Get-DomainObjectAcl -SamAccountName student1 -ResolveGUIDs
```

- Get the ACLs associated with the specified prefix to be used for search:
```powershell
Get-DomainObjectAcl -SearchBase "LDAP://CN=DomainAdmins,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local" -ResolveGUIDs -Verbose
```

- We can also enumerate ACLs using ActiveDirectory module but without resolving GUIDs:
```powershell
(Get-Acl 'AD:\CN=Administrator,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local').Access
```

- Search for interesting ACEs:
```powershell
Find-InterestingDomainAcl -ResolveGUIDs
```

- Get the ACLs associated with the specified path:
```powershell
Get-PathAcl -Path "\\dcorp-dc.dollarcorp.moneycorp.local\sysvol"
```

## Trusts

### Domain Trust mapping

- Get a list of all domain trusts for the current domain
```powershell
Get-DomainTrust
Get-DomainTrust -Domain us.dollarcorp.moneycorp.local
```

```powershell
Get-ADTrust
Get-ADTrust -Identity us.dollarcorp.moneycorp.local
```

### Forest mapping

- Get details about the current forest:
```powershell
Get-Forest
Get-Forest -Forest eurocorp.local
```
```powershell
Get-ADForest
Get-ADForest -Identity eurocorp.local
```

- Get all domains in the current forest:
```powershell
Get-ForestDomain
Get-ForestDomain -Forest eurocorp.local
(Get-ADForest).Domains
```

- Get all global catalogs for the current forest:
```powershell
Get-ForestGlobalCatalog
Get-ForestGlobalCatalog -Forest eurocorp.local
Get-ADForest | select -ExpandProperty GlobalCatalogs
```

- Map trusts of a forest (no Forest trusts in the lab):
```powershell
Get-ForestTrust
Get-ForestTrust -Forest eurocorp.local
Get-ADTrust -Filter 'msDS-TrustForestTrustInfo -ne "$null"'
```

