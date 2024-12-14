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

## Virtual Machine 
- **Operation System**: Ubuntu x86_64
---

## Steps Taken
### What I Did
**Analyzed the `vuln.c` Program**:
   I read the program file that the professor gave to us to get a better understanding how the code functioned and the best way to approach this. It was easy to understand because the professor gave us the code which he had commented and the best way to tackle this was through the  `strcpy(buffer, str); ` because it had a buffer overflow problem. In the main function it will make a large array which is bigger than the buffer which will read in from a file named "badfile" which we will create in the exploit program. This is slightly diffeent than a regular buffer overflow because the professor had asked us to do it with the use of a reverse shell. In a reverse shell it will allow the payload to allow the attacker to remotely connect to the targeted machine. This is done to the use of the payload which contains the reverse shellcode which opens a network connection. This is done by :
   - Creating a socket.
   - Connects to the attackers Ip address and port.
   - provides us with a remote shell.

### Disabling measure:
In this lab we awere asked to this in an older model which had a linux kernel of 2.4 but i was curious if i could have accomplished doing this lab with a modern linux. IN order to have the best p[ossible chance to do this i needed to disable as much of the defenses possible and I did this by typing `sudo sysctl -w kernel.randomize_va_space=0`.

To compile the vulnerable program `vuln.c`, I used the following command:
`gcc -g -z execstack -fno-stack-protector vuln.c -o vuln` and in this lab i attempted to see the difference of trying it in both 64 and also in 32-bit. I did this by entering `gcc -g -m32 -z execstack -fno-stack-protector vuln.c -o vuln`.


### Explanation of Compilation Flags

1. **`-g`**:
- Includes debugging information for use with `gdb`.
- Provides readable symbols for debugging.

2. **`-z execstack`**:
- Enables an executable stack.
- Modern systems mark the stack as non-executable by default, preventing attacks like stack smashing. This flag allows executing shellcode injected during the exploit.

3. **`-fno-stack-protector`**:
- Disables the stack canary, a security feature that prevents buffer overflow attacks.
- Without this flag, the program terminates if a stack overflow is detected, preventing overwriting the return address.

4.**`-m32`**:
- This is used to target and make it to choose the 32 architecture and not the default one which is 64.
- Is also used to limit the program to 32-bit registers.

---
## Staring of

I created the badfile by using `touch badfile` this will be alter be done witht the use of exploit.c. When running the vuln code I first need it to cause a segmentation fault to get a better understanding with the memory. I need to find the offset and a return address. I tested it out and figures that the input size needed to cause a segmentation fault was 112. This is when I later realized that the last four bits were 0x42424242 which was the last four letters that were after my A's. When i did this I later realized that when i checked `info registers` that I had also realized that i overwrote the RIP. Rips is needed to be overwritten because it is the next instruction pointer this will allow us to take control of what is going to happen next. This well be used when implementing the NOP sled. THe following shows the rgisters were I overwrote RIP.

Info Register:
![Overwrote Screenshot](Overwrote.png)


The use of the NOP sled well allow it to reach the shellcode that I had injected into the buffer which would be in between the NOP. I wanted to see were my $rsp started and did the following to find it. The nope sled will be used to continue like a sled until it finally reaches my shellcode. THe shellcode compared to the regular overflow in thios lab we will be targeting the rever shellcode so we need to gain access remotely this is done by the computer we are targeting to have a port listening and the information needed to attack will be implemented and inside the shellcode. Since we are doing this locally all we need to do is have a port open and listsning and two terminals running.




## Debugging Code

- Used `gdb` to examine the program’s memory layout:
- Located the buffer's starting address with `p &buffer`.
- Found the saved base pointer with `p $rsp`.
- Calculated the offset to the return address with `p/d`. I used this to find the difference betwerrn the rsp and the buffer.



4. **Tested the Exploit**:
- Compiled the vulnerable program and payload:
  ```
  gcc -fno-stack-protector -z execstack vuln.c -o vuln
  gcc exploit.c -o exploit
  ```
- Ran the exploit:
  ```
  ./vuln < badfile
  ```

### Why It Failed

1. **ASLR (Address Space Layout Randomization)**:
- Memory addresses were randomized, making it difficult to reliably overwrite the return address.
- ASLR was disabled during testing but could still cause issues if not fully disabled system-wide.

2. **Stack Protection**:
- Stack canaries prevented the program from executing arbitrary code.
- Stack protection was disabled during compilation, but further issues could arise.

3. **Improper information**:
4. - THe reason why it may have failed is beacuse we werent using the currect address or the right amont of offset even though i was able to overwrite RIP.

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
- Fine-tune the shellcode to match the target environment. This means making sure that my code has the correct shellcode and the offset needed.


---




