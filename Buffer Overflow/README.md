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

#include <stdio.h>
#include <stdbool.h>
#include <string.h>

void success() {

//Prints success message
	printf("\n*************\nYOU GUESSED CORRECTLY :)\n*************\n");

}

bool compareWord(){

  //Declares variable for a 10 character-long word
	char guess[10];

	printf("\nGuess the word: \n");

  //Receives the word
  gets(guess);

//Compares user input to the correct word and returns a bool indicating if the user guessed (1) or not (0)
if (strcmp(guess,"hello") == 0){
		return 1;
	}
	return 0;
}


int main(int argc, char *argv[]){

  //Prints the sentence with two newlines (puts by default adds a newline at the end)
	puts("----Word guesser----\n");

 //Stores whether the user guessed or not in a variable
	bool guessedCorrectly = compareWord();

  //If the user guessed correctly, prints the success message. If not, asks the user to try again
	if(guessedCorrectly == 1) success(); else puts("\nYou guessed wrong. Try again...\n");

}


```
In order to make it vulnerable, I used the ```gets()``` function, which doesn't validate the input size before storing it. Given this vulnerability, this function is actually deprecated in newer C versions, and even in the older version I used, compiling it issued the following warning:

  ![Implicit declaration warning](/Images/implicitDecWarn.png)

In order to carry out this exercise: 
  -  Compilation.- ```gcc```; I particularly used the flag for executing with stack protection (```--fstack-protector```), which terminates the process with an error if it detects a buffer overflow
  -  Visualization of the binary's assembly code.- ```objdump```
  -  Execution.- ```qemu-x86_64```, which is the processor I worked with


  
  ### - Exploiting the vulnerability

Knowing how the stack is organized is relevant to really understand this attack.

When a function is called, the first things the compiler does is copy the return address at the top of the stack (so it knows where to return to after the function ends) and the current position to a register (frame pointer). Then, it separates the rest of the stack's memory locations depending on the bytes the function will occupy. 

* It's worth noting when I say _memory_ I'm referring to registers (temporary memory locations in the CPU)

In ```compareWord```'s case, the stack separated 48 bytes of memory (0x30 in hexadecimal):

  ![Prologue](/Images/prologue.png)

The first three lines of this image show what I described a couple paragraphs ago. The fourth line is the canary being generated.

This canary is later stored at rbp-0x08, where rbp is the frame pointer, which doesn't change throughout the execution. The reason it's subtracting is that in x86_64 processors, memory is filled downwards (from higher memory to lower), so the canary would be 0x08 positions lower than the frame pointer. This is relevant because we're talking about dynamic memory, meaning each execution will result in different memory addresses, so knowing an object's relative position is useful to pinpoint its exact location during each execution.

![Canary and Buffer](/Images/canaryAndBuff.png)

In the third line, the buffer is being declared.


To find out the distance between the canary and the buffer, I went to the area where ```gets``` was being called and noticed the address that was being calculated for the buffer at rbp-0x12. 

![Buffer address](/Images/bufferAddr.png)

To calculate the canary's distance to the buffer, I subtracted their distance to rbp: 0x12 - 0x08 = 18 bytes - 8 bytes = 10 or 0xa.

This is an oversimplified view of how the stack looks:

![Stack doigram](/Images/stackDiagram.png)


Additionally, I added some code to the C program in order to visualize the current execution'e addresses for ```success()```, the canary, and the frame pointer.


With all this information, I could begin the exploit.

I first created a temporary fifo file, which allow data exchange between processes. This left the program running:

![Executing word guesser](/Images/fstackExec.png)


From a different terminal, I used echo to send a payload to the fifo file. Because I was using the stack protection mode, I had to bypass the canary if I wanted to access the success message. To do this, the payload had to contain the amount of bytes between the canary and the buffer, the canary's contents, and the desired return address (success).

This was the result. As can be observed, the canary remained the same, yet I was able to obtain the success message:

![Exploit result](/Images/guessed.png)

* The line under "Guess the word" was added in the code to visualize the frame pointer's address, so I didn't write anything directly to the program.


This was compiled with the ```-fno-stack-protector``` flag, and we can observe how the frame changes after the overflow, which is something the canary tries to prevent:

![Without stack protector](/Images/fno-stackExec.png)


  ### - Patching the vulnerability

In order to patch this vulnerability, I used ```fgets()``` instead of ```gets()```. This function actually validates that the input has the size it's supposed to.

Now it doesn't matter which input is sent; it'll only read up to the predefined number of bytes.

![Patched Output](/Images/patched.png)

I also compiled with the stack protection flag so it has extra validation.


## How to prevent

To prevent this vulnerability from being exploited, the first thing to do is ensuring every accepted input has clear limits regarding size and type of data (the latter to prevent other attacks such as injections). A

Avoiding the use of functions that don't check inputs (like ```gets()```) is also relevant.


## Conclusions

With this exercise I learned how the compiler manages memory, how buffer overflows work, and the danger they pose.

Knowing about this vulnerability and how it can be exploited is relevant because it'll help me recognize more easily when it is occurring (or when there's a script that performs this type of attack) so that it can be mitigated/stopped before it causes damage. It is also useful to know when developing scripts so we can implement safety measures.


## Sources

- Oliveira, D. M. (2026b). Heavy Wizardry 101: Shellcodes, Backdoors, Droppers, and Worms. No Starch Press.
  
- What is buffer overflow? Attacks, types & vulnerabilities | Fortinet. (n.d.). Fortinet. https://www.fortinet.com/resources/cyberglossary/buffer-overflow
