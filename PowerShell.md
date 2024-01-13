# Windows Enumeration
### Handy PowerShell Stuff

1. List all processes:

`Get-Process`

`Get-Acl` - lists the access control list to a path, which can be given with the -Path flag. Also pipe the output into Format-List or (FL) to prettify:
`Get-Acl -Path \Path\To\File | fl`

2. The path for a running process:

`(Get-Process -Name [proc-name]).path`

3. List all installed applications:

```PowerShell
# List 32-bit
Get-Process
"HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname

# List 64-bit
Get-Process
"HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname
```


4. Encode & Decode Base64:

```PowerShell
# Encode
$string = 'Hello World'
$bytes = [System.Text.Encoding]::UTF8.GetBytes($string)
[Convert]::ToBase64String($bytes);
# ---- mind the semicolon ----
# Decode
$string = [Convert]::FromBase64String('SGVsbG8gd29ybGQ=');
[System.Text.Encoding]::UTF8.GetString($string);
```

5. Checking for permissions on a path:
```PowerShell
icacls C:\path\of\choice
```

6. Checking for running services:
```PowerShell
Get-CimInstance -ClassName wi32_service | Select Name, State, PathName | Where-Object {$_.State -like 'Running'}

# Should blurt out all the running processes. Feel free to do more powershell magic to narrow search. 
```

7. Controlling Services:
```PowerShell
Start-Service [service_name]
Stop-Service [service_name]
Restart-Service [service_name]
```

8. Path:
```PowerShell
$env:path
```

9. PowerShell Foreach for Pinging:
```PowerShell
$ipAddresses = @("192.168.1.1", "192.168.1.2", "192.168.1.3", "192.168.1.4", "192.168.1.5")

foreach ($ip in $ipAddresses) {
    $result = Test-Connection $ip -Count 1 -Quiet

    if ($result) {
        Write-Host "$ip is reachable."
    } else {
        Write-Host "$ip is not reachable."
    }
}
```

10. Resolve SID to Name
```PowerShell
Convert-SidToName S-1-5-21-1987370270-658905905-1781884369-553
```

11. Filtering with the powershell '-eq' operator:
```PowerShell
Get-Command | ? {$_.PropertyNameOfGetCommandOutput -eq "Value_of_Object"} | do something;
# That is the syntax for filtering.
```

12. Tasklist Command
```PowerShell
tasklist | findstr 'calc' # enter a prcess name to find the running process.
```