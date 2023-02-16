Masscan es una herramienta que nos ayudará a llevar a cabo [[Host discovery]] a una gran red mediante un barrido de los puertos más comunes.

## Uso

```bash
masscan -p22,80,443,445,... -Pn --rate=8000 --banners <máscara de red> -e <interfaz de red> --router-ip <router ip>
```

