---
title: Invisi-Shell
---
# ¿Qué es?

Es una herramienta para bypasear la seguridad de [[CRTP/PowerShell|PowerShell]].

The tool hooks the .NET assemblies (`System.Management.Automation.dll` and `System.Core.dll`) to bypass logging. It uses a *CLR Profiler API* to perform the hook.

> "A common language runtime (CLR) profiler is a dynamic link library (DLL) that consists of functions that receive messages from, and send messages to, the CLR by using the profiling API. The profiler DLL is loaded by the CLR at run time."

# Instalación

# Uso

With admin privileges:
```powershell
RunWithPathAsAdmin.bat
```

With non-admin privileges:
```powershell
RunWithRegistryNonAdmin.bat
```

Type `exit` from the new PowerShell session to complete the clean-up.

