#pwn #ctf 

# Reconocimiento

Nos dan un binario. Si lo descompilamos con [[ghidra]]:

```c
undefined4 main(undefined1 param_1)
{
  uint local_20;
  uint local_1c;
  uint local_18;
  int local_14;
  undefined1 *local_10;
  
  local_10 = &param_1;
  puts("");
  puts("");
  puts("                                                                             ");
  puts("                          #                mmmmm  mmmmm    \"    mm   m   mmm ");
  puts("  mmm    mmm    mmm    mmm#          mmm   #   \"# #   \"# mmm    #\"m  # m\"   \"");
  puts(" #   \"  #\"  #  #\"  #  #\" \"#         #   \"  #mmm#\" #mmmm\"   #    # #m # #   mm");
  puts(
      "  \"\"\"m  #\"\"\"\"  #\"\"\"\"  #   #          \"\"\"m  #      #   \"m   #    #  # # #    #"
      );
  puts(" \"mmm\"  \"#mm\"  \"#mm\"  \"#m##         \"mmm\"  #      #    \" mm#mm  #   ##  \"mmm\"");
  puts("                                                                             ");
  puts("");
  puts("");
  puts("Welcome! The game is easy: you jump on a sPRiNG.");
  puts("How high will you fly?");
  puts("");
  fflush(stdout);
  local_18 = time((time_t *)0x0);
  srand(local_18);
  local_14 = 1;
  while( true ) {
    if (0x1e < local_14) {
      puts("Congratulation! You\'ve won! Here is your flag:\n");
      get_flag();
      fflush(stdout);
      return 0;
    }
    printf("LEVEL (%d/30)\n",local_14);
    puts("");
    local_1c = rand();
    local_1c = local_1c & 0xf;
    printf("Guess the height: ");
    fflush(stdout);
    __isoc99_scanf(&DAT_00010c9a,&local_20);
    fflush(stdin);
    if (local_1c != local_20) break;
    local_14 = local_14 + 1;
  }
  puts("WRONG! Sorry, better luck next time!");
  fflush(stdout);
                    /* WARNING: Subroutine does not return */
  exit(-1);
}
```

Vemos que se van generando valores entre 0 y 15 (`0xf`) con *rand* al que se le pasa la hora actual como *seed*. Debemos acertar 30 veces.

# Explotación

Para la explotación, simplemente debemos correr un programa que empiece justo a la vez (tendrá la misma seed), y vaya generando valores:

```c
#include <stdio.h>
#include <time.h>
#include <stdlib.h>

int main (){
    int i;
    
    srand(time(0));
    for (i = 0; i < 30; i++){
        
        printf("%d\n", rand() & 0xf); 
    }
    
    return 0;
}
```

Para que tenga la misma hora debemos correrlo a la vez:

```bash
./solve | nc jupiter.challenges.picoctf.org 34558
```

Y ya estaría resuelto!
