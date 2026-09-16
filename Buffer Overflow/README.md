# Buffer Overflow

#### _** Disclaimer: the purpose of this document is to show what I learned regarding buffer overflows in order to understand them and be capable of defending against them. This document doesn't contain any exploits**_

## What is a buffer overflow?

A buffer overflow is a vulnerability where more data is passed to the temporary area of memory where the response will be stored than what it was expecting. To further understand how it works and its relevance, I'll explain other important concepts.

## Why does it happen?

Whenever a variable that requires multiple spaces (such as an array or string) is defined, the compiler allocates a range of memory addresses for the variable to go exclusively; that is, the sole purpose of this memory range is to contain the variable and it is known as a __buffer__. 

If an attacker were to overload the buffer (pass more bytes than it can handle), these bytes could overwrite memory addresses outside its scope.

According to Fortinet, the most common type of buffer overflow is stack-based, and that's the one I'll be focusing on in this document.

At a low-level, in order to keep track of function calls, the compiler separates a region of the memory and starts allocating instructions and data pertaining to said functions there. This memory area is known as a __stack__. When the functions ends, the stack is destroyed, the compiler manages the return values depending on the instructions provided, and it goes back to the return address (memory address that points to the next instructions to execute after the function). 

With this in mind, if a buffer overload were to occur and the attacker has knowledge of the memory range in relation to the return address, they could potentially craft a payload that can cause an overload and modify the return address so it returns the data they want regardless of the instructions/code. This is what I did for this exercise.

Before jumping to the walkthrough, a concept that's relevant to understand is the __canary__. In order to try mitigating this vulnerability, some compilers allow compilation using stack protection. What this does is randomly separate a memory address and save it before the buffer limits (on architectures like x86_64 where the stack grows downwards, the canary would be at a higher position). Before the function ends, the compiler checks this memory address and compares it to the saved one. If they're different, the function is terminated.

While the canary mitigates this vulnerability, it's still not entirely safe, as I'll demonstrate in the next section.

## Basic Buffer Overflow

  ### - Vulnerable code

I generated a basic script in C that asks the user to guess a word. If it is correct, the program congratulates the user; if not, it tells the user to try again.

``` c

#include <>

```
  
  ### - Exploit flowchart
  ### - Patching the vulnerability
  
## How to prevent

## Conclusions

With this exercise I learned how the compiler manages memory, how buffer overflows work, and the danger they pose.

Knowing about this vulnerability and how it can be exploited is relevant because it'll help me recognize more easily when it is occurring (or when there's a script that performs this type of attack) so that it can be mitigated/stopped before it causes damage. It is also useful to know when developing scripts so we can implement safety measures.

## Sources

- Oliveira, D. M. (2026b). Heavy Wizardry 101: Shellcodes, Backdoors, Droppers, and Worms. No Starch Press.
  
- What is buffer overflow? Attacks, types & vulnerabilities | Fortinet. (n.d.). Fortinet. https://www.fortinet.com/resources/cyberglossary/buffer-overflow
