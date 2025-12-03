---
layout: post
title:  "Код в фильмах: Кракен 2025"
date:   2025-12-03 00:00:00
categories: [movie, code]
---

![]({{site.url}}/images/kraken-movie/cover.webp)

Не собирался смотреть этот фильм, но надысь Баженов выложил на него обзор, и на одном из кадров мелькнул фрагмент, по очертаниям очень похожий на ассемблерный код. Пришлось ознакомиться.

Весь фильм, то тут, то там показывают разного рода мониторы, но ничего конкретного на них не разобрать. Жуткое мыло.

Но в конце, в ключевой сцене таки случилось интересное.

Сцена запуска мандулы "супер-оружия":<br>
![]({{site.url}}/images/kraken-movie/001.webp)

Внизу чётко виден ассемблерный код. Это фрагмент из [учебника по программированию](https://web.archive.org/web/20250317015153/https://www.tutorialspoint.com/assembly_programming/assembly_registers.htm). Рисуем на экране девять звёздочек в ряд.


```asm
   mov	edx,9    ;message length
   mov	ecx,s2   ;message to write
   mov	ebx,1    ;file descriptor (stdout)
   mov	eax,4    ;system call number (sys_write)
   int	0x80     ;call kernel

   mov	eax,1    ;system call number (sys_exit)
   int	0x80     ;call kernel

section	.data
msg db 'Displaying 9 stars',0xa ;a message
len equ $ - msg  ;length of message
s2 times 9 db '*'
```

После такой годной зацепки становится возможным разобрать мыльные фрагменты.

Сцена с таймером самоуничтожения:<br>
![]({{site.url}}/images/kraken-movie/002.webp)


Фрагмент с кодом очень размытый:<br>
![]({{site.url}}/images/kraken-movie/003.webp)

но, благодаря подсказке в предыдущей сцене, его получается идентифицировать. Слева код из того же учебника, вычисляет и выводит на экран [сумму чисел](https://web.archive.org/web/20250306111025/https://www.tutorialspoint.com/assembly_programming/assembly_arithmetic_instructions.htm) 3 и 4.
А справа код чуть сложнее: вычисляется [сумма чисел](https://web.archive.org/web/20250314191736/https://www.tutorialspoint.com/assembly_programming/assembly_numbers.htm) 12345 и 23456 в [двоично-десятичном](https://ru.wikipedia.org/wiki/%D0%94%D0%B2%D0%BE%D0%B8%D1%87%D0%BD%D0%BE-%D0%B4%D0%B5%D1%81%D1%8F%D1%82%D0%B8%D1%87%D0%BD%D1%8B%D0%B9_%D0%BA%D0%BE%D0%B4) представлении.

```asm
section	.text
   global _start    ;must be declared for using gcc

_start:             ;tell linker entry point
   mov	eax,'3'
   sub     eax, '0'

   mov 	ebx, '4'
   sub     ebx, '0'
   add 	eax, ebx
   add	eax, '0'

   mov 	[sum], eax
   mov	ecx,msg
   mov	edx, len
   mov	ebx,1	;file descriptor (stdout)
   mov	eax,4	;system call number (sys_write)
   int	0x80	;call kernel

   mov	ecx,sum
   mov	edx, 1
   mov	ebx,1	;file descriptor (stdout)
   mov	eax,4	;system call number (sys_write)
   int	0x80	;call kernel

   mov	eax,1	;system call number (sys_exit)
   int	0x80	;call kernel

section .data
   msg db "The sum is:", 0xA,0xD
   len equ $ - msg
   segment .bss
   sum resb 1
```


```asm
section	.text
   global _start        ;must be declared for using gcc

_start:	                ;tell linker entry point

   mov     esi, 4       ;pointing to the rightmost digit
   mov     ecx, 5       ;num of digits
   clc
add_loop:
   mov 	al, [num1 + esi]
   adc 	al, [num2 + esi]
   aaa
   pushf
   or 	al, 30h
   popf

   mov	[sum + esi], al
   dec	esi
   loop	add_loop

   mov	edx,len	        ;message length
   mov	ecx,msg	        ;message to write
   mov	ebx,1	        ;file descriptor (stdout)
   mov	eax,4	        ;system call number (sys_write)
   int	0x80	        ;call kernel

   mov	edx,5	        ;message length
   mov	ecx,sum	        ;message to write
   mov	ebx,1	        ;file descriptor (stdout)
   mov	eax,4	        ;system call number (sys_write)
   int	0x80	        ;call kernel

   mov	eax,1	        ;system call number (sys_exit)
   int	0x80	        ;call kernel

section	.data
msg db 'The Sum is:',0xa
len equ $ - msg
num1 db '12345'
num2 db '23456'
sum db '     '
```

Теперь можно попробовать по смутным очертаниям догадаться, что там было в середине фильма, в сцене, где герои пытались деактивировать своё "супер-оружие":<br>
![]({{site.url}}/images/kraken-movie/004.webp)

Тут сложнее! Так просто не деактивировать! Слева [вычисляем факториал числа 3](https://web.archive.org/web/20250127124916/https://www.tutorialspoint.com/assembly_programming/assembly_recursion.htm). А в правом фрагменте - пример [модификации строк в памяти](https://web.archive.org/web/20250127124837/https://www.tutorialspoint.com/assembly_programming/assembly_addressing_modes.htm) на примере замены имени: "Zara" -> "Nuha". Программа выводит "Zara Ali Nuha Ali".<br>
Да, авторы учебника - индусы.

```asm
section	.text
   global _start         ;must be declared for using gcc

_start:                  ;tell linker entry point

   mov bx, 3             ;for calculating factorial 3
   call  proc_fact
   add   ax, 30h
   mov  [fact], ax

   mov	  edx,len        ;message length
   mov	  ecx,msg        ;message to write
   mov	  ebx,1          ;file descriptor (stdout)
   mov	  eax,4          ;system call number (sys_write)
   int	  0x80           ;call kernel

   mov   edx,1            ;message length
   mov	  ecx,fact       ;message to write
   mov	  ebx,1          ;file descriptor (stdout)
   mov	  eax,4          ;system call number (sys_write)
   int	  0x80           ;call kernel

   mov	  eax,1          ;system call number (sys_exit)
   int	  0x80           ;call kernel

proc_fact:
   cmp   bl, 1
   jg    do_calculation
   mov   ax, 1
   ret

do_calculation:
   dec   bl
   call  proc_fact
   inc   bl
   mul   bl        ;ax = al * bl
   ret

section	.data
msg db 'Factorial 3 is:',0xa
len equ $ - msg

section .bss
fact resb 1
```

```asm
section	.text
   global _start     ;must be declared for linker (ld)
_start:             ;tell linker entry point

   ;writing the name 'Zara Ali'
   mov	edx,9       ;message length
   mov	ecx, name   ;message to write
   mov	ebx,1       ;file descriptor (stdout)
   mov	eax,4       ;system call number (sys_write)
   int	0x80        ;call kernel

   mov	[name],  dword 'Nuha'    ; Changed the name to Nuha Ali

   ;writing the name 'Nuha Ali'
   mov	edx,8       ;message length
   mov	ecx,name    ;message to write
   mov	ebx,1       ;file descriptor (stdout)
   mov	eax,4       ;system call number (sys_write)
   int	0x80        ;call kernel

   mov	eax,1       ;system call number (sys_exit)
   int	0x80        ;call kernel

section	.data
name db 'Zara Ali '
```

А в сцене стрельбы из этого оружия теперь становится понятно, что вычисления нужны те же, что и для самоуничтожения: посчитать сумму 3 и 4. И сложить 12345 с 23456.<br>
![]({{site.url}}/images/kraken-movie/007.webp)
