.NET lacks security features implemented in PowerShell

# Challenges

1. detections by AV, EDR, etc
2. deliver of payload (as opposed to PowerShell)
3. detection by logging like process creation, cli logging, etc


## AV Bypass

mainly focus on bypass of signature-based detection by Defender
- use DefenderCheck to verify signatures

can use code obfuscation to bypass
- [Codecepticon](https://github.com/Accenture/Codecepticon)
	- needs to be compiled in Visual Studio
- not on student VM, so i don't think it will be needed on the exam

- can use the following to obfuscate source code with Codecepticon
	- `C:\AD\Tools\Codecepticon.exe --action obfuscate --module csharp --verbose --path "C:\AD\Tools\Rubeus-master\Rubeus.sln" --map-file "C:\AD\Tools\Rubeus-master\mapping.html" --profile rubeus --rename ncefpavs --rename-method markov --markov-min-length 3 --markov-max-length 10 --markov-min-words 3 --markov-max-words 5 --string-rewrite --string-rewrite-method xor`

[ConfuserEx](https://mkaring.github.io/ConfuserEx/)
- free .NET obfuscator
- can stop AVs from performing signature detections


