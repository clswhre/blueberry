---
title: "Trying Reverse Engineering & Crackmes"
date: 2026-06-21 11:00:00 +0300
tags: [ghidra, ltrace, c-programming, static-analysis, dynamic-analysis, crackme]
---

During the Summer School I wrote about earlier, we were told that it's possible to join the university's CTF team (OSINT, web, *reverse*, etc.). That caught my interest, so with some background in C and Assembly, I decided to dive into it.

## Case 1: Static Analysis & Writing a Keygen (Ghidra + C)

The first task was to analyze a binary that required entering a valid license key based on a username. I used **Ghidra** for this.

### Analyzing the Logic
After loading the binary into Ghidra, I found the `main` function. Its logic turned out to be classic:
1. The program checks the number of command-line arguments (`argc == 2`), expecting a nickname.
2. An internal function `get_license` is called to generate the key.
3. The password entered by the user is compared against the generated one.

The real magic was hiding in the generation function. The program used a hardcoded alphabet (a character string) and a math algorithm based on the modulo operation (`%`) to assemble the correct key character-by-character for a specific user:
* Alphabet: `QAZPLWSXOKMEYDCIJNRFVUHBTGqpalzmwoeirutyskdjfhgxncbv1750284369`
* Formula: `(i + username[i]) % 62`
* Key length: 24 characters.

### My Own Keygen
Once I understood the algorithm, I decided to write a full keygen in C.

Here's my code:

```c
#include <stdio.h>
#include <string.h>

int main(){
    char nickname[32] = {0}; 
    char generated_key[266] = {0};
    char idk_key[256] = "QAZPLWSXOKMEYDCIJNRFVUHBTGqpalzmwoeirutyskdjfhgxncbv1750284369";
    int i;
    
    printf("Enter nickname: ");
    scanf("%31s", nickname);
    
    for(i = 0; i < 24; i++){
        generated_key[i] = idk_key[(i + nickname[i]) % 62];
    }
    generated_key[i] = '\0';
    
    printf("key for <%s> : %s\n", nickname, generated_key);
    return 0;
}
```

The program compiles successfully and works flawlessly.

---

## Case 2: Dynamic Analysis & a Quick Win with ltrace

A crackme called "The Wired" by author OpenWiredSource. The `file` utility showed it's a 64-bit ELF binary for Linux, stripped of debug symbols.

On a normal run with a random password (e.g. `flag`), the program created an image called `lain_is_here.png` (from the anime "Serial Experiments Lain") in the current directory and printed `[-] Incorrect Password`.

I decided to try a dynamic approach (`ltrace`). If the program checks the flag using standard library functions like `strcmp`, the password can be intercepted right in memory during execution.

I ran the binary through the `ltrace` utility:
```bash
$ ltrace ./thewired flag
fopen("lain_is_here.png", "wb")                         = 0x55fa168b8010 
fwrite("\211PNG\r\n\032\n", 1, 177508, 0x55fa168b8010)  = 177508
fclose(0x55fa168b8010)                                  = 0
strcmp("flag", "we_all_love_serial_experiments_l"...)   = -17
puts("[-] Incorrect Password"[-] Incorrect Password)    = 23
+++ exited (status 0) +++
```

Interception result:

The terminal output clearly shows the system calls. First the program created the image, then a call to `strcmp()` happened. The program itself kindly placed the correct password into the register for comparison against my invalid input, and `ltrace` successfully captured it. The full password turned out to be `we_all_love_serial_experiments_lain`.

Running the program with this password gave: ACCESS GRANTED. Welcome to The Wired. Everyone is connected.
