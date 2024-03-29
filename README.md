# black-square
An assembly language program recreating Kazimir Malevich's Black Square with a pixel image

## What is this?
Started as a joke, now becoming reality and I attemt to render an image in the assembly languge. I know nothing about assembly but see this as a coding challenge where I build upon resouces from the web, that I document here

## First steps: "Hello World!"
I am working on MacOSX so my choice fell on using nasm (https://www.nasm.us/) as my assembler ``brew install nasm``. After that  I went to the terminal and created a folder with a ``hello.asm`` file in it.

### x86 nasm assembly code
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
	; I think $ represents the current adress of the stacks start
	; subtracting message_length from that will offset it by 13 positions
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

### Building the object code file
``nasm -f macho hello_world.asm``

### Link the "Hello World!" program
``ld -static -o hello_world.o -e start hello_world.o``

### Run the executable program 
``./hello_world.o``

### Console output
```sh
➜  HelloWorld ./hello_world.o 
Hello World!
```

## After setting up this program I aim for the next steps
Pulling first an example code from this stack overflow discussion (https://stackoverflow.com/questions/10345125/how-to-open-and-draw-an-image-in-assembly) from Rommel Samanez. I haven't compiled it yet, but if this works I should be able to simplify it to write just a black square image. Tried compiling and failed. Maybe it supports a different system architecture than I have. I'm not sure but it uses ``elf64`` as a target (https://en.wikipedia.org/wiki/Executable_and_Linkable_Format) which is different from the ``hello.asm`` using the ``macho`` as target. Reading more on Wikipedia and let this go for the moment.  
```assembly
; Program to draw lines to 1000x1000 Bitmap File
; Compile with:
;     nasm -f elf64 -o drawLines64.o drawLines64.asm
; Link with:
;     ld -m elf_x86_64 -o drawLines64 drawLines64.o
; Run with:
;     ./drawLines64
;==============================================================================
; Author : Rommel Samanez
; https://github.com/rsamanez/assemblerx64
;==============================================================================

global _start


section .data
  fileName:  db "lines.bmp",0
  fileFlags: dq 0102o         ; create file + read and write mode
  fileMode:  dq 00600o        ; user has read write permission
  fileDescriptor: dq 0
  bitmapImage: times 3000000 db 0
section .rodata    ; read only data section
    bmpWidth   equ  1000
    bmpHeight   equ  1000
    bmpSize equ 3 * bmpWidth * bmpHeight
    headerLen equ 54
    header: db "BM"         ; BM  Windows 3.1x, 95, NT, ... etc.
    bfSize: dd bmpSize + headerLen   ; The size of the BMP file in bytes
            dd 0                 ; reserved
    bfOffBits: dd headerLen   ; The offset, i.e. starting address, of the byte where the bitmap image data (pixel array) can be found
    biSize:   dd 40    ; header size, min = 40
    biWidth:  dd bmpWidth
    biHeight: dd bmpHeight
    biPlanes:       dw 1     ; must be 1
    biBitCount:     dw 24    ; bits per pixel: 1, 4, 8, 16, 24, or 32
    biCompression:  dd 0     ; uncompressed = 0
    biSizeImage:    dd bmpSize  ; Image Size - may be zero for uncompressed images
    hresolution:    dd 0
    vresolution:    dd 0
    palettecolors:  dd 0
    importantcolors: dd 0

section .text

; drawPixel(x,y,color{RGB}) ( R8,R9,R10) 
drawPixel:
    push rax
    push rbx
    ;---------------------------------
    mov rax,3000
    mul r9
    push rax
    mov rax,3
    mul r8
    pop rbx
    add rax,rbx         ; offset:  rax = 3*x+3000*y
    push r10
    pop rbx
    mov word[bitmapImage+rax],bx
    shr rbx,16
    mov byte[bitmapImage+rax+2],bl
    ;----------------------------------
    pop rbx
    pop rax
    ret
; drawLine(x1,y1,x2,y2,color) color = R10  x1=r8 y1=r9 x2=r11 y2=r12    
drawLine:
    push rax
    push rbx
    push rcx                  ;  line   y = M*x/1000   M=w*1000/q  w=y2-y1  q=x2-x1 
    push rdx
    ;-----------------------------------
    mov rax,r12             ; rax <- y2
    sub rax,r9              ; rax <- y2 - y1
    mov rbx,1000
    mul rbx                 ; rax <- rax*1000
    mov rbx,r11             ; rbx <- x2
    sub rbx,r8              ; rbx <- x2 - x1
    xor rdx,rdx             ; rdx = 0
    div rbx                 ; RAX / RBX  RAX= value  M
    push rax
    pop rcx                 ; rcx <- M
    mov rbx,r8              ; rbx <- x1
    ;-----------------------------------------
    push r8
    push r9
nextPixel:
    mov rax,rbx             ; rbx = x
    mul rcx                 ; rax = M*x
    push rcx
    xor rdx,rdx
    mov rcx,1000
    div rcx                 ; rax = M*x/1000
    pop rcx
    mov r8,rbx              ; r8 <- x
    mov r9,rax              ; r9 <- y
    call drawPixel
    inc rbx
    cmp rbx,r11
    jnz nextPixel
    pop r9
    pop r8
    ;-----------------------------------
    pop rdx
    pop rcx
    pop rbx
    pop rax
    ret


drawLines:
    mov r8,0
    mov r9,0
    mov r10,0xff0000        ; color RED
    mov r11,900
    mov r12,900
    call drawLine
    ret  

_start:
    mov rax,2                 ;   sys_open
    mov rdi,fileName          ;   const char *filename
    mov rsi,[fileFlags]       ;   int flags
    mov rdx,[fileMode]        ;   int mode
    syscall
    mov [fileDescriptor],rax

    ; write the header to file
    mov rax,1                 ; sys_write
    mov rdi,[fileDescriptor]
    mov rsi,header
    mov rdx,54
    syscall

    call drawLines

    ; write the Image to file
    mov rax,1                 ; sys_write
    mov rdi,[fileDescriptor]
    mov rsi,bitmapImage
    mov rdx,3000000
    syscall

    ; close file Descriptor
    mov rax,3                 ; sys_close
    mov rdi,[fileDescriptor]
    syscall

    ; EXIT to OS  sys_exit
    mov rax,60
    mov rdi,0
    syscall
```

## Things I would need in general
### .data section
* Store the filename (db) "square.bmp" 
* Store the length of the filename (equ) $-filename 
* Store the value of "Black" (Not sure how, but in general it should be 0x00000000?) 
* Store the total ammount of pixels as a integer number, should be 1 for a 1x1 pixel bmp
* Store the magic number of a bmp file 
* Store all of the other header infos that belong in a bmp file
* Store the SYS call to open and write to a file

### .text section
* Create a process that writes a single black pixel to a x, y position in an image
* Create a process that combines what I need
* * Push the length of the filename on the stack
* * Push the char buffer of the filename on the stack
* * Push 1 - not sure why
* * Call my pixel process?
* Close the file
* Clean up the mess and move the $ stack pointer back to it's value (Means I've to calculate how many times I pushed something on the stack add all of those together and add this to th ``esp``(Stack base pointer)
* Push a 0 on the first postion of the stack
* Push the SYS command to exit (1) on the stack
* Interrupt the execution of my program (int 0x80)
* Exit the program


## Research links
* A Mandelbrot set image in wasm (Web Assembly) on a html page
* https://medium.com/@alexc73/programming-using-web-assembly-c4c73a4e09a9
