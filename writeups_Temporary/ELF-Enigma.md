# ELF Enigma

**Points**: 500  
**Solves**: 0  
**Flag Format**: HPTU{___}

## Challenge Description

A compact ELF binary hides its secret in the decompiled code; find the exact input that makes the program print "success".

## Solution

1. **Binary Analysis**: Used Ghidra to decompile the ELF binary and analyze the `main` function.
  ```c
  undefined8 main(void)
  
  {
    int iVar1;
    char local_28 [32];
    
    fwrite("Password: ",1,10,stdout);
    __isoc99_scanf("DoYouEven%sCTF",local_28);
    iVar1 = strcmp(local_28,"__dso_handle");
    if ((-1 < iVar1) && (iVar1 = strcmp(local_28,"__dso_handle"), iVar1 < 1)) {
      printf("Try again!");
      return 0;
    }
    iVar1 = strcmp(local_28,"_init");
    if (iVar1 == 0) {
      printf("Correct!");
    }
    else {
      printf("Try again!");
    }
    return 0;
  }
  ```
3. **Input Logic**: Discovered the main program uses `scanf("DoYouEven%sCTF", buffer)` after `fwrite("Password: ",1,10,stdout)` which expects input in a specific format.
4. **Password Extraction**: The actual comparison was against the string `"_init"`, but due to the scanf format, the full input needed to be `DoYouEven_init`.
5. **Validation**: The program compares the extracted string (content between "DoYouEven" and "CTF") with `"_init"`.
6. **Flag:** `HPTU{DoYouEven_init}`

## Key Insight

The scanf format string "DoYouEven%sCTF" required wrapping the actual password _init with "DoYouEven".
