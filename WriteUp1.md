# Lab Writeup for Buffer Overflow Project
Francisco Diaz
Due Date: December 13 2024

## Goal
The goal is to gain root access by the use of reverse shell by exploiting the buffer.

## Files Given by Professor:
- vuln.c

## Files needed to be submitted:
- Exploit.c
- Writeup.md

---

## Virtual Machine Setup
- **Operation System**: Ubuntu x86_64
- **Disabled**: I turned off the address space layout randomization security mechanism by `sudo sysctl -w kernel.randomize_va_space=0`.

---

## Creating the Executable

To compile the vulnerable program `vuln.c`, I used the following command:
`gcc -g -z execstack -m32 -fno-stack-protector vuln.c -o vuln`

### Explanation of Flags
1. **`-g`**:
   - The `-g` flag includes debugging information in the executable because without it it would not have any readable symobols while I am debugging which is needed.
   - This is important for analyzing the program with `gdb`, as it provides access to symbolic debugging data.

2. **`-z execstack`**:
   - The `-z execstack` flag enables an executable stack.
   - Modern systems often mark the stack as non-executable to prevent attacks like stack smashing which we are doing on this lab. This will allow the stack to execute code, which is necessary for executing shellcode injected during the exploit.

3. **`-m32`**:
   - The `-m32`  forces the compilation to target a 32-bit architecture.
   - The exploit depends on specific memory layout assumptions and shellcode designed for 32-bit systems. Compiling as 64-bit (default on x86_64 systems) would result in mismatched memory addressing and prevent the exploit from working.

4. **`-fno-stack-protector`**:
   - This flag disables the stack canary, a security feature added by default to protect against buffer overflow attacks.
   - Without this flag, the program would terminate if a stack overflow is detected, making it impossible to overwrite the return address.

---
## Steps Taken
### What I Did
1. **Analyzed the `vuln.c` Program**:
   - Identified the buffer overflow vulnerability caused by the use of `gets()`.
   - Used `gdb` to examine the program’s memory layout:
     - Found the buffer's starting address with `p &buffer`.
     - Located the saved base pointer with `p $ebp`.
     - Calculated the offset to the return address with `p/d`.

2. **Constructed the Payload**:
   - Created a unique pattern manually since i was not able to use cylic pattern or pwn on my device.
   - Built a payload with:
     - A **NOP sled** to ensure reliable execution.
     - Placeholder shellcode to spawn `/bin/sh`.
     - Overwrote the return address with the calculated buffer address.

3. **Tested the Exploit**:
   - Compiled the vulnerable program and payload with:
     ```
     gcc -m32 -fno-stack-protector -z execstack vuln.c -o vuln
     gcc -m32 exploit.c -o exploit
     ```
   - Ran the exploit:
     ```
     ./vuln < badfile or ./vuln
     ```

### Why It Failed
- **ASLR (Address Space Layout Randomization)**:
  - Memory addresses were randomized, making it difficult to reliably overwrite the return address.
  - ASLR was disabled during testing but could still cause issues if not fully disabled system-wide.

- **Stack Protection**:
  - The presence of `canary` values in the stack prevented the program from executing arbitrary code.
  - Stack protection was disabled during compilation, but further issues may have arisen.

---

## What I Would Do to Succeed
To perform the attack successfully:
1. **Disable All Protections**:
   - Ensure ASLR is fully disabled:
     ```
     echo 0 > /proc/sys/kernel/randomize_va_space
     ```
   - Verify stack protection is disabled by using `checksec` on the binary.

2. **Adjust Payload**:
   - Fine-tune the shellcode to match the target environment.
   - Use `pwntools` or similar tools to dynamically locate the buffer and return address.

3. **Use a Controlled Environment**:
   - Run the exploit in a virtual machine with all security mechanisms disabled.





