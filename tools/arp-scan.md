Nos permite realizar [[Host discovery]] mediante *ARP*.

## Instalación

```bash
sudo apt install arp-scan
```

## Uso

Simplemente debemos especificar la interfaz y la red (en este caso la red local):

```bash
sudo arp-scan -I wlp3s0 --localnet
```
