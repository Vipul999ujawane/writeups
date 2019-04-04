## Crack Me 1
### Category : Reversing

We are given a stripped binary file. Let's open it in Ghidra

Function worth watching:

``` 

undefined8 FUN_00101179(void)

{
  long in_FS_OFFSET;
  char local_28;
  char local_27;
  char local_26;
  char local_25;
  char local_23;
  char local_22;
  char local_20;
  char local_1f;
  char local_1e;
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  printf("Enter The Secret Code To Open the Vault: ");
  fgets(&local_28,0x14,stdin);
  printf("\nFlag: ");
  if (local_27 == 'D') {
    printf("en");
    if (local_26 == 'D') {
      printf("cryptCTF{BYE}");
                    /* WARNING: Subroutine does not return */
      exit(0);
    }
    if (local_26 == 1) {
      printf("cry");
      if (local_25 != 'A') {
                    /* WARNING: Subroutine does not return */
        exit(0);
      }
      printf("ptC");
      if (local_23 != ' ') {
                    /* WARNING: Subroutine does not return */
        exit(0);
      }
      printf("TF{");
      if (local_22 != '!') {
                    /* WARNING: Subroutine does not return */
        exit(0);
      }
      printf("gdb");
      if (local_20 != 'e') {
                    /* WARNING: Subroutine does not return */
        exit(0);
      }
      printf("_or");
      if (local_1f != 0x19) {
                    /* WARNING: Subroutine does not return */
        exit(0);
      }
      printf("_r2?");
      if (local_1e != '\t') {
                    /* WARNING: Subroutine does not return */
        exit(0);
      }
      puts("}");
    }
  }
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return 0;
}
```

I did not bother with passing the rules and just copied the values from the respective printf's to get the FLAG : ```encryptCTF{gdb_or_r2?}```