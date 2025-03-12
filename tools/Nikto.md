---
title: Nikto
---
# ¿Qué es?

Es una herramienta que permite escanear un servicio web.

# Instalación

```bash
sudo apt update && sudo apt install -y perl
git clone https://github.com/sullo/nikto
cd nikto/program
chmod +x ./nikto.pl
```

# Uso

```bash
nikto -h inlanefreight.com -Tuning b
```