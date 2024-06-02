Es una colección de clases de python para trabajar con protocolos de red. Está enfocado en dar un acceso de bajo nivel a los paquetes y, para algunos protocolos, al protocolo en sí mismo. 

## Instalación

La herramienta se encuentra en el siguiente repositorio de [github](https://github.com/fortra/impacket). Para instalarlo, lo clonamos y ejecutamos el siguiente comando:

```shell
git clone https://github.com/fortra/impacket
```
```shell
cd impacket  
pip3 install .
# OR:  
sudo python3 setup.py install
```
```shell
# En caso de que falte algún módulo:  
pip3 install -r requirements.txt
```

## mssqlclient.py

Este script nos permitirá conectarnos a un servidor de base de datos con el servicio *mssql*. Se encuentra en `impacket/examples/`:

```shell
cd /impacket/examples/
```
```shell
python3 mssqlclient.py -h #Para mostrar la ayuda
```

Un ejemplo mostrado en la máquina [[Archetype HTB]] sería:

```shell
python3 mssqlclient.py ARCHETYPE/sql_svc@{TARGET_IP} -windows-auth
```

Con el que nos conectamos a un host `ARCHETYPE` con el usuario `sql_svc` utilizando `-windows-auth` para que utilice la autenticación de windows.