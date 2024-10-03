# ¿Qué es?

# Usos

## Ver siguientes instrucciones

```gdb
x/10i $pc
```
> Muestra las siguientes 10 instrucciones

## Ver datos

### Ver el stack

```gdb
x/20x $rsp
```
> Para ver 20 palabras en el stack

### Examinar un registro

Para ello utilizamos la *x*:

```gdb
x/x $r13
x/s $r13
x/7x $r13
```