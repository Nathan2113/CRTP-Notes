Tools for complete coverage on Local Privilege Escalation
[PowerUp](https://github.com/PowerShellmafia/PowerSploit/tree/master/Privesc)
[Privesc](https://github.com/itm4n/PrivescCheck)
[winPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS)

Various ways of locally escalating privileges
- missing patches
- automated deployment and AutoLogon passwords
- AlwaysInstallElevated (any user can run MSI as SYSTEM)
- misconfigured services
- DLL Hijacking and DLL Sideloading
- Kerberos and NTLM relaying

Services issues using PowerUp
- Get services with unquote paths and a space in their name
	- `Get-ServiceUnquoted -Verbose`
![[Pasted image 20260131023520.png]]
- for Abyss Web Server, if we drop Abyss.exe in `C:\WebServer` and restart the service, the computer will run Abyss.exe since that will be chosen before Abyss Web Server (because of the spaces)
- Get services where the current user can write to its binary path or change arguments
	- `Get-ModifiableServiceFile -Verbose
	- `Get-ModifiableService -Verbose`
		- Get services whose configuration current user can modify
		- `sc.exe sdshow <binary>` to see file permissions
![[Pasted image 20260131023854.png]]
- WD (everyone) has the same permissions as BA (Admins)
- not opsec-safe


Run all checks from
- PowerUp
	- `Invoke-AllChecks`
- Privesc
	- `Invoke-PrivEscCheck`
- PEASS-ng
	- `winPEASx64.exe`