## The Governor's Secret
### Category : Reverse Engineering

So, we have a 64-bit ELF executable. 

On running we get

```
┌──(greenpanda999㉿zacian)-[~/Desktop/Hax/gov]
└─$ ./chal
Usage: chal password
```

We gotta input a password.

```
┌──(greenpanda999㉿zacian)-[~/Desktop/Hax/gov]
└─$ ./chal ARC{sdsds} 
Nope.
```

On opening it in Ghidra, we can see that it's a stripped binary. After some basic reversing, I find the `main()`.

```

undefined8 main(int param_1,long param_2)

{
  undefined8 uVar1;
  int iVar2;
  int iVar3;
  int iVar4;
  int iVar5;
  long in_FS_OFFSET;
  char local_118 [264];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  if (param_1 < 2) {
    puts("Usage: chal password\n");
                    /* WARNING: Subroutine does not return */
    exit(-1);
  }
  uVar1 = *(undefined8 *)(param_2 + 8);
  iVar2 = FUN_001012e9(uVar1);
  iVar3 = FUN_0010157e(uVar1);
  iVar4 = FUN_0010185a(uVar1);
  iVar5 = FUN_00101b7e(uVar1);
  if ((((iVar2 == 0) || (iVar3 == 0)) || (iVar4 == 0)) || (iVar5 == 0)) {
    puts("Try again!");
  }
  else {
    printf("Woot! The flag is %s\n",uVar1);
    snprintf(local_118,0x100,
             "openssl enc -d -aes-256-cbc -pbkdf2 -in secret_encrypted.txt -k \"%s\"",uVar1);
    system(local_118);
  }
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return 0;
}
```

We need to pass 4 condition checks to get to the "correct" condition or else we get a `Try again!`.

But on running the binary, we got a `Nope.`.

Probably a fault in one of the upper checks failing.

Thing is, `Nope.` has no xrefs in the current binary. Maybe something else is happening?

So, on further reversing I can every function has this :

```
00101308 e8 00 00 00 00        CALL       LAB_0010130d
                            LAB_0010130d 
0010130d 58                    POP        RAX
```

Calling the next address and pop-ing the stack, allows RAX to hold the value in the instruction pointer.

The exectuable is trying to locate "itself" within the binary. A popular technique in packed malwares.

So, for the above function, I put a break point near the `mprotect()` function call, and then start moving forward. Eventually, I can notice the code section of the function start changing, clearly the binary is getting unpacked. 

Using `gdb`, once I find the function has been unpacked, I dump the "new" binary contents and start to analyze that.

Based on this new binary I find, 

1. What the function does.

2. The unpacking is same for each function

3. After each function processes the input, the function packs itself.


So, I need to create a new dump for each function.

1. Func1 => checks for input length of 0x17

2. Func2 =>   

```
uVar9 = (ulong)((((cVar1 == 'A' && cVar2 == 'R') && cVar3 == 'C') && cVar4 == '{') && cVar5 == '}'
                 );
```

3. I processed the third function dynamically as it checked each character and compared with an internal function

4. The last function was a bit different. I found this magic number in there. `0xefcdab89` which is a part of md5. So I assumed a hash check is being done. 

I find `MD5(lyfe)=855eaad768495f4066f26e7e625113fa`.


## Flag 

```
┌──(greenpanda999㉿zacian)-[~/Desktop/Hax/gov]
└─$ ./chal ARC{Obfusc4ti0n_4_lyfe}
Woot! The flag is ARC{Obfusc4ti0n_4_lyfe}

[START OF TRANSMISSION]

I must be brief lest I'm caught sending you this. 
The governor's favorite drink is...

....AAARGGHHHHHH! 

[END OF TRANSMISSION]
```