## Crackme 3
### Category : Reversing

This was a rather interesting and slightly long challenge to solve.
We are given a binary. Let's check it out in Ghidra. The one function which looks like the main.

```

undefined4 FUN_00011502(void)

{
  undefined4 uVar1;
  int in_GS_OFFSET;
  int local_70;
  code *local_68 [4];
  code *local_58;
  undefined local_54 [64];
  int local_14;
  undefined *local_10;
  
  local_10 = &stack0x00000004;
  local_14 = *(int *)(in_GS_OFFSET + 0x14);
  local_68[0] = FUN_00011286;
  local_68[1] = FUN_000112bd;
  local_68[2] = FUN_000112e6;
  local_68[3] = FUN_00011392;
  local_58 = FUN_00011410;
  setvbuf(stdout,(char *)0x0,2,0);
  puts("Hi!, i am a BOMB!\nI will go boom if you don\'t give me right inputs");
  local_70 = 0;
  while (local_70 < 5) {
    printf("Enter input #%d: ",local_70);
    __isoc99_scanf(&DAT_000120b9,local_54);
    (*local_68[local_70])(local_54);
    local_70 = local_70 + 1;
  }
  FUN_0001122d();
  uVar1 = 0;
  if (local_14 != *(int *)(in_GS_OFFSET + 0x14)) {
    uVar1 = FUN_00011670();
  }
  return uVar1;
}
```
So, we have a array of function pointers which are called in succession if they are returned.

We need a total of 5 passwords to get the FLAG.

For the first function :

```

void FUN_00011286(char *param_1)

{
  int iVar1;
  
  iVar1 = FUN_000115fc();
  iVar1 = strcmp(param_1,(char *)(iVar1 + 0xd8e));
  if (iVar1 != 0) {
    FUN_00011258();
  }
  return;
}

```
Not much info here. Using Strace, we get that input is compared with   ` CRACKME02 `

For the second password :

```

void FUN_000112bd(int *param_1)

{
  FUN_000115fc();
  if (*param_1 != -0x21524111) {
    FUN_00011258();
  }
  return;
}

```
Also, the disassembly :
```
                             **************************************************************
                             *                          FUNCTION                          *
                             **************************************************************
                             undefined FUN_000112bd(undefined4 param_1)
             undefined         AL:1           <RETURN>
             undefined4        Stack[0x4]:4   param_1                                 XREF[1]:     000112d4(R)  
             undefined4        Stack[-0x10]:4 local_10                                XREF[2]:     000112cd(W), 
                                                                                                   000112d9(R)  
                             FUN_000112bd                                    XREF[4]:     FUN_00011502:00011533(*), 
                                                                                          FUN_00011502:00011539(*), 
                                                                                          000120f0, 0001220c(*)  
        000112bd 55              PUSH       EBP
        000112be 89 e5           MOV        EBP,ESP
        000112c0 83 ec 18        SUB        ESP,0x18
        000112c3 e8 34 03        CALL       FUN_000115fc                                     undefined FUN_000115fc()
                 00 00
        000112c8 05 38 2d        ADD        EAX,0x2d38
                 00 00
        000112cd c7 45 f4        MOV        dword ptr [EBP + local_10],0xdeadbeef
                 ef be ad de
        000112d4 8b 45 08        MOV        EAX,dword ptr [EBP + param_1]
        000112d7 8b 00           MOV        EAX,dword ptr [EAX]
        000112d9 39 45 f4        CMP        dword ptr [EBP + local_10],EAX
        000112dc 74 05           JZ         LAB_000112e3
        000112de e8 75 ff        CALL       FUN_00011258                                     undefined FUN_00011258()
                 ff ff
                             LAB_000112e3                                    XREF[1]:     000112dc(j)  
        000112e3 90              NOP
        000112e4 c9              LEAVE
        000112e5 c3              RET
```

There's a simple compare of the input with ` 0xdeadbeef`

For the third password :

```
/* WARNING: Function: __i686.get_pc_thunk.bx replaced with injection: get_pc_thunk_bx */

void FUN_000112e6(int param_1)

{
  size_t sVar1;
  int in_GS_OFFSET;
  uint local_2c;
  undefined4 local_25;
  undefined4 local_21;
  undefined4 local_1d;
  undefined4 local_19;
  undefined4 local_15;
  undefined local_11;
  int local_10;
  
  local_10 = *(int *)(in_GS_OFFSET + 0x14);
  local_25 = 0x7479585a;
  local_21 = 0x66396255;
  local_1d = 0x6538376c;
  local_19 = 0x794a6776;
  local_15 = 0x4e4a4b33;
  local_11 = 0;
  local_2c = 0;
  while( true ) {
    sVar1 = strlen((char *)&local_25);
    if (sVar1 <= local_2c) break;
    if (*(char *)((int)&local_25 + local_2c) != *(char *)(param_1 + local_2c)) {
      FUN_00011258();
    }
    local_2c = local_2c + 1;
  }
  if (local_10 != *(int *)(in_GS_OFFSET + 0x14)) {
    FUN_00011670();
  }
  return;
}
```

We have a simple compare of the string with a hardcoded value. 

For the fourth password :

```

void FUN_00011392(char *param_1)

{
  size_t sVar1;
  int iVar2;
  
  sVar1 = strlen(param_1);
  if (3 < sVar1) {
    FUN_00011258();
  }
  iVar2 = atoi(param_1);
  if ((iVar2 * iVar2 * 2 - iVar2) * 2 + -3 + iVar2 * iVar2 * iVar2 == 0) {
    puts("SUBSCRIBE TO PEWDIEPIE");
  }
  else {
    FUN_00011258();
  }
  return;
}
```
Here the input is converted to a int and then passed on to solved a quadratic equation.

The equation has a root 1.36, since it is an int, we can pass "1" to get to the next stage

For the last part :

```

/* WARNING: Function: __i686.get_pc_thunk.bx replaced with injection: get_pc_thunk_bx */

void FUN_00011410(char *param_1)

{
  int in_GS_OFFSET;
  char local_1a;
  char local_19;
  char local_18;
  char local_17;
  char local_16;
  char local_15;
  char local_14;
  char local_13;
  char local_12;
  int local_10;
  
  local_10 = *(int *)(in_GS_OFFSET + 0x14);
  strncpy(&local_1a,param_1,10);
  puts("Validating Input 4");
  if ((int)local_12 + (int)local_1a == 0xd5) {
    if ((int)local_13 + (int)local_19 == 0xce) {
      if ((int)local_14 + (int)local_18 == 0xe7) {
        if ((int)local_15 + (int)local_17 == 0xc9) {
          if (local_16 == 'i') {
            puts("you earned it");
          }
        }
        else {
          FUN_00011258();
        }
      }
      else {
        FUN_00011258();
      }
    }
    else {
      FUN_00011258();
    }
  }
  else {
    FUN_00011258();
  }
  if (local_10 != *(int *)(in_GS_OFFSET + 0x14)) {
    FUN_00011670();
  }
  return;
}

```
10 chars to input are copied to range of chars. A easy buffer overflow. Then the sums of first and last value is added and compared to a constant. Since the constant is > 256. we  have to divide the val by two and pass values accordingly.

The final exploit 
```
from pwn import *
print('CRACKME02\n' + p32(0xdeadbeef)+'\n'+p32(0x7479585a)+''+p32(0x66396255)+''+p32(0x6538376c)+''+p32(0x794a6776)+''+p32(0x4e4a4b33)+'\n'+'1\n'+'\x6b\x67\x74\x65i\x64\x73\x67\x6A\n')
```
We get the flag 

```
vipul@ubuntu:~$ cat reverse | nc 104.154.106.182 7777
Hi!, i am a BOMB!
I will go boom if you don't give me right inputs
Enter input #0: Enter input #1: Enter input #2: Enter input #3: SUBSCRIBE TO PEWDIEPIE
Enter input #4: Validating Input 4
you earned it
encryptCTF{B0mB_D!ffu53d}
```