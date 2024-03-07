---
title: Keeper HTB
date: 2023-12-09
---
#linux 

## Fase de reconocimiento

Comenzamos comprobando que la máquina se encuentra activa con `ping`, y en base al *ttl* (63), sospechamos que se trata de una máquina **linux**. A continuación tratamos de buscar puertos abiertos con [[nmap]]:

```shell
sudo nmap -p- --open -sS --min-rate 5000 -n -Pn <ip>
```
```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

Vemos que hay dos puertos abiertos, el [[22-ssh]] y el [[80-http]], por lo que parece que la intrusión será vía web. Vamos a volver a lanzar [[nmap]] para detectar versiones y servicios:

```bash
nmap -p22,80 -sCV -n -Pn <ip>
```
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

No vemos nada relevante, por lo que accedemos a la web. Una vez dentro, vemos un enlace que nos redirige a `http://tickets.keeper.htb/rt/`. En esta ruta encontramos un **panel de login**.

## Explotación

Aunque vemos que se está utilizando un servicio llamado *request tracker* en su versión 4.4.4, no parece haber nada interesante. En cambio probando las credenciales por defecto del servicio (**root:password**), ganamos acceso.

En el panel del admin podemos encontrar entre otras cosas otro usuario del sistema, **`lnorgaard`** y su contraseña **`Welcome2023!`**. Si tratamos de acceder por [[22-ssh]], conseguimos entrar y vemos la *flag de usuario* (`9cdbd07e4ae50157ee4426be935caf08`).

## Escalada de privilegios

