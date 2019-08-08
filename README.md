# black-square
An assembly language program recreating Kazimir Malevich's Black Square with a pixel image

# What is this?
Started as a joke, now becoming reality and I attemt to render an image in the assembly languge. I know nothing about assembly but see this as a coding challenge where I build upon resouces from the web, that I document here

# First steps
I am working on MacOSX so my choice fell on using nasm (https://www.nasm.us/) as my assembler ``brew install nasm``. After that  I went to the terminal and created a folder with a ``hello.asm``file in it.
```assembly
; Hello World in assembly language
; Matthias Jäger 
;
; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ;
; Static data and constants
; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ;
section .data
	; adress = bytes of textual data as ASCII hexadecimal, linebreak	
	message db "Hello World!", 0x0a 
	; calculate the length of the text based on adresses
	message_length equ $-message
; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ;
; Program logic
; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ; ;
section .text
global start
start 
	; #define	SYS_write          4
	; Write the number 4 in the eax register 4 stands for a
	; system call to SYS_write
	mov eax, 4
	; Stack nbytes, buf, fd
	; A dword is 4 byte sized number on the stack 
	; After each push call the stack register is reduces by the number 4
	; First push the number representing the length of the message
	push dword message_length
	; Then push the message buffer on the stack
	push dword message
	;fd	
	push dword 1
	; Fill up the stack to a total of 16 bytes
	push dword eax
	; interrupt the processor 
	; Jump to system call
	int 0x80
	; Clean up and take the esp register back to it's original value
	add esp, 16
	; Exit the program with syscall 1
	; #define	SYS_exit           1
	mov eax, 1
	; Error code 0	
	push dword 0
	int 0x80
```

## Building the object code file
``sh nasm -f macho hello_world.asm``

# Compile the "Hello World!" program
``sh ld -static -o hello_world.o -e start hello_world.o``

# Run the program 
``sh ./hello_world.o``

# Outputs
``
➜  HelloWorld ./hello_world.o 
Hello World!
```
