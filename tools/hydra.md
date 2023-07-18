Es una herramienta para hacer ataques de fuerza bruta.

# Uso

### Ftp

```bash
hydra -l <usuario> -P <wordlist (rockyou.txt)> ftp://<ip> -t <n_hilos>
```