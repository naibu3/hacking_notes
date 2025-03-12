# ¿Qué es?

Es el emulador de terminal de Windows.

# Uso

## Decodificar b64

```powershell-session
[IO.File]::WriteAllBytes("C:\Users\Public\id_rsa", [Convert]::FromBase64String("<HASH=>"))
```