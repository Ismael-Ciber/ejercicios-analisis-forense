# Ejercicios de reversing

## Ejercicio 1

### - Compila el código fuente `hello.c` y ejecuta el programa. ¿Qué hace? ¿Qué salida produce?

```bash
┌──(kali㉿kali)-[~/Desktop/reversing-binarios]
└─$ gcc hello.c -o hello
                                                                                                                                                             
┌──(kali㉿kali)-[~/Desktop/reversing-binarios]
└─$ ./hello                    
Hola, mundo!
```
Tan solo imprime en pantalla `Hola, mundo!`

### - Analiza el ejecutable con `objdump` y `strings`. ¿Qué instrucciones se utilizan para imprimir la cadena? ¿Qué cadenas aparecen en el ejecutable?

```
0000000000001139 <main>:
    1139:       55                      push   %rbp
    113a:       48 89 e5                mov    %rsp,%rbp
    113d:       48 8d 05 c0 0e 00 00    lea    0xec0(%rip),%rax        # 2004 <_IO_stdin_used+0x4>
    1144:       48 89 c7                mov    %rax,%rdi
    1147:       e8 e4 fe ff ff          call   1030 <puts@plt>    <--------------------
    114c:       b8 00 00 00 00          mov    $0x0,%eax
    1151:       5d                      pop    %rbp
    1152:       c3                      ret
```

En la función `main`, se llama a la función `puts@plt`, que a su vez, contiene estas intrucciones para imprimir en pantalla:

```
0000000000001030 <puts@plt>:
    1030:       ff 25 ca 2f 00 00       jmp    *0x2fca(%rip)        # 4000 <puts@GLIBC_2.2.5>
    1036:       68 00 00 00 00          push   $0x0
    103b:       e9 e0 ff ff ff          jmp    1020 <_init+0x20>

```

Para ver que cadenas aparecen en el ejecutable, es muy facil de visualizar con strings: 

```
└─$ strings hello    
           
/lib64/ld-linux-x86-64.so.2
puts
__libc_start_main
__cxa_finalize
libc.so.6
GLIBC_2.2.5
GLIBC_2.34
_ITM_deregisterTMCloneTable
__gmon_start__
_ITM_registerTMCloneTable
PTE1
u+UH
Hola, mundo!   <------------------------------
;*3$"
GCC: (Debian 13.3.0-3) 13.3.0
Scrt1.o
```


### - Analiza el ejecutable con radare2, obten las direcciones de las funciones usadas.

```
┌──(kali㉿kali)-[~/Desktop/reversing-binarios]
└─$ r2 -A hello
```

```
[0x00001050]> afl
0x00001030    1      6 sym.imp.puts
0x00001040    1      6 sym.imp.__cxa_finalize
0x00001050    1     33 entry0
0x00001080    4     34 sym.deregister_tm_clones
0x000010b0    4     51 sym.register_tm_clones
0x000010f0    5     54 sym.__do_global_dtors_aux
0x00001130    1      9 sym.frame_dummy
0x00001154    1      9 sym._fini
0x00001139    1     26 main
0x00001000    3     23 sym._init
[0x00001050]> 
```

### - Analiza la funcion `main`, ¿que puedes obtener a partir de la misma?

```
[0x00001050]> pdf @ main
            ; DATA XREF from entry0 @ 0x1064(r)
┌ 26: int main (int argc, char **argv, char **envp);
│           0x00001139      55             push rbp
│           0x0000113a      4889e5         mov rbp, rsp
│           0x0000113d      488d05c00e..   lea rax, str.Hola__mundo_   ; 0x2004 ; "Hola, mundo!"
│           0x00001144      4889c7         mov rdi, rax                ; const char *s
│           0x00001147      e8e4feffff     call sym.imp.puts           ; int puts(const char *s)
│           0x0000114c      b800000000     mov eax, 0
│           0x00001151      5d             pop rbp
└           0x00001152      c3             ret
[0x00001050]> 
```

- `push rbp` El programa guarda una marca para recordar dónde empezó esta parte del código. Es como decir: "recuerda este punto para luego poder volver".

- `mov rbp, rsp` Establece una base para trabajar con la memoria del programa. Piensa en esto como preparar un espacio de trabajo.

- `lea rax, [0x2004]` El programa busca la frase `"Hola, mundo!"` que está guardada en otra parte y guarda su dirección (no la frase, solo dónde está) en el registro de 64 bits llamada `rax`.

- `mov rdi, rax` Almacena el contenido del registro `rax` en otro registro de 64 bits `rdi` del cual la función llamada leerá el primer argumento.

- `call puts` Llama a una función especial del sistema que muestra el texto en la pantalla. En este caso, va a imprimir `"Hola, mundo!"`.

- `mov eax, 0` Le dice al sistema que todo ha ido bien. El `0` es como un "OK".

- `pop rbp` Recupera la marca que guardamos al principio, para volver atrás con seguridad.

- `ret` Termina esta parte del programa y vuelve al sistema o al que lo llamó.



## Ejercicio 2

### - Analiza el ejecutable `serial-check` y obtén acceso a la aplicación. 

Podemos comprobar que es un ejecutable para linux

```
┌──(kali㉿kali)-[~/Desktop/reversing-binarios]
└─$ file serial-check 
serial-check: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=6da4b5417edeac02867024006afbcc1d2026689b, for GNU/Linux 3.2.0, not stripped
```

Al ejecutar el programa, nos pide una contraseña, la cual no sabemos

```
┌──(kali㉿kali)-[~/Desktop/reversing-binarios]
└─$ ./serial-check

Introduce la contraseña: hola
Acceso denegado.
                                                                                                                                                             
┌──(kali㉿kali)-[~/Desktop/reversing-binarios]
└─$ ./serial-check
Introduce la contraseña: admin
Acceso denegado.
                                                                                                                                                             
┌──(kali㉿kali)-[~/Desktop/reversing-binarios]
└─$ ./serial-check
Introduce la contraseña: test
Acceso denegado.
```

Al analizarlo con `strings` podemos observar a simple vista cual es la contraseña

```
┌──(kali㉿kali)-[~/Desktop/reversing-binarios]
└─$ strings serial-check

/lib64/ld-linux-x86-64.so.2
puts
__stack_chk_fail
__libc_start_main
__cxa_finalize
printf
__isoc99_scanf
strcmp
libc.so.6
GLIBC_2.7
GLIBC_2.4
GLIBC_2.2.5
GLIBC_2.34
_ITM_deregisterTMCloneTable
__gmon_start__
_ITM_registerTMCloneTable
PTE1
u+UH
Introduce la contrase
%19s
bombardinocrocodilo    <----------------
Acceso concedido.
Acceso denegado.
9*3$"
GCC: (Ubuntu 13.3.0-6ubuntu2~24.04) 13.3.0
Scrt1.o
__abi_tag
crtstuff.c
```

Al poner la contraseña correcta, podemos acceder

```
┌──(kali㉿kali)-[~/Desktop/reversing-binarios]
└─$ ./serial-check      
Introduce la contraseña: bombardinocrocodilo
Acceso concedido.
```

## Ejercicio 3

### - Modifica el ejecutable de `serial-check` para que acepte el serial `pass1234`.

Abrimos el ejecutable en modo lectura con radare2

```
┌──(kali㉿kali)-[~/Desktop/reversing-binarios]
└─$ r2 -w serial-check

WARN: Relocs has not been applied. Please use `-e bin.relocs.apply=true` or `-e bin.cache=true` next time
[0x000010e0]> 
```

Buscamos las cadenas de texto con `iz`

```
[0x000010e0]> iz
[Strings]
nth paddr      vaddr      len size section type  string
―――――――――――――――――――――――――――――――――――――――――――――――――――――――
0   0x00002004 0x00002004 25  27   .rodata utf8  Introduce la contraseña:
1   0x0000201f 0x0000201f 4   5    .rodata ascii %19s
2   0x00002024 0x00002024 19  20   .rodata ascii bombardinocrocodilo
3   0x00002038 0x00002038 17  18   .rodata ascii Acceso concedido.
4   0x0000204a 0x0000204a 16  17   .rodata ascii Acceso denegado.
[0x000010e0]> 
```

Ubicamos la contraseña en la dirección de memoria `0x00002024` y accedemos a ella

```
[0x000010e0]> s 0x00002024
[0x00002024]> 
```

Para establecer la nueva contraseña, deberemos de pasarlo a formato hexadecimal

```
┌──(kali㉿kali)-[~]
└─$ echo -n "prprpatapim" | xxd -p

707270727061746170696d
```

Establecemos el cambio en la dirección de memoria donde se almacena la contraseña agregando `00` al final de la contraseña en hexadecimal para establecer donde se acaba el string, lo que se le conoce como un byte nulo, si cambias una cadena como "bombardinocrocodilo" por "prprpatapim" sin el \x00, y hay datos después en memoria, el programa puede seguir leyendo bytes basura y comparar mal la contraseña.

Comprobamos que está todo correcto imprimiendo el valor de la dirección de memoria y guardamos los datos con `q`

```
[0x00002024]> wx 707270727061746170696d00
[0x00002024]> ps 12 @ 0x00002024
prprpatapim\x00
[0x00002024]> q

```

Ahora al intentar acceder con la nueva contraseña `prprpatapim` tendremos acceso concedido y si lo intentamos con la antigua `bombardinocrocodilo` nos encontramos con el acceso denegado

```
┌──(kali㉿kali)-[~/Desktop/reversing-binarios]
└─$ ./serial-check         
Introduce la contraseña: prprpatapim
Acceso concedido.
                                                                                                                                                             
┌──(kali㉿kali)-[~/Desktop/reversing-binarios]
└─$ ./serial-check
Introduce la contraseña: bombardinocrocodilo
Acceso denegado.
```

## Ejercicio 4

### - Analiza los ejecutables `mindreader` y `mindreader2`. ¿Eres capaz de saber cual está compilada desde C, y cual desde python? ¿Por qué?

Ambos ejecutables utilizan el compilador `gcc`, típico de C, esto nos inidica que el ejecutable de pyhton ha usado herramientas como: PyInstaller, Nuitka, cx_Freeze.
Estas herramientas empaquetan el intérprete de Python junto con al script. Internamente, incluyen código C generado automáticamente (por eso r2 detecta "GCC" como compilador).

```
┌──(kali㉿kali)-[~/Desktop/reversing-binarios]
└─$ r2 -d mindreader   
WARN: Relocs has not been applied. Please use `-e bin.relocs.apply=true` or `-e bin.cache=true` next time
[0x7f68f26d8b00]> iI
arch     x86
baddr    0x561da632c000
binsz    14477
bintype  elf
bits     64
canary   true
injprot  false
class    ELF64
compiler GCC: (Ubuntu 13.3.0-6ubuntu2~24.04) 13.3.0  <---------
crypto   false
endian   little
```

```
┌──(kali㉿kali)-[~/Desktop/reversing-binarios]
└─$ r2 -d mindreader2
WARN: Relocs has not been applied. Please use `-e bin.relocs.apply=true` or `-e bin.cache=true` next time
[0x7f59ae931b00]> iI
arch     x86
baddr    0x400000
binsz    7439348
bintype  elf
bits     64
canary   true
injprot  false
class    ELF64
compiler GCC: (Ubuntu 7.5.0-3ubuntu1~18.04) 7.5.0   <-------
crypto   false
endian   little
havecode true
```

Usamos `iz` para ver las cadenas, también podemos usar `iz~py` para buscar indicios de algo relacionado con pyhton.

> Nota el output que nos da al usar `iz` en el ejecutable mindreader2 es de más de 300 cadenas distintas, no lo añado para no abultar este documento, añado el resultado tras filtrar las cadenas que contienen `py`

```
┌──(kali㉿kali)-[~/Desktop/reversing-binarios]
└─$ r2 -d mindreader 
WARN: Relocs has not been applied. Please use `-e bin.relocs.apply=true` or `-e bin.cache=true` next time
[0x7f2c4bb67b00]> iz
[Strings]
nth paddr      vaddr          len size section type  string
―――――――――――――――――――――――――――――――――――――――――――――――――――――――――――
0   0x00002004 0x564a49cc3004 7   8    .rodata ascii manzana
1   0x0000200c 0x564a49cc300c 7   8    .rodata ascii naranja
2   0x00002014 0x564a49cc3014 5   7    .rodata utf8  limón
3   0x0000201b 0x564a49cc301b 7   9    .rodata utf8  plátano
4   0x00002024 0x564a49cc3024 6   7    .rodata ascii cereza
5   0x0000202b 0x564a49cc302b 5   6    .rodata ascii fresa
6   0x00002031 0x564a49cc3031 6   8    .rodata utf8  sandía
7   0x00002039 0x564a49cc3039 5   7    .rodata utf8  melón
8   0x00002040 0x564a49cc3040 4   5    .rodata ascii pera
9   0x00002045 0x564a49cc3045 4   5    .rodata ascii kiwi
10  0x0000204a 0x564a49cc304a 10  11   .rodata ascii Password:
11  0x00002057 0x564a49cc3057 18  19   .rodata ascii La flag es NOTNULL
12  0x0000206a 0x564a49cc306a 15  16   .rodata ascii Acceso denegado
[0x7f2c4bb67b00]> 
```

```
┌──(kali㉿kali)-[~/Desktop/reversing-binarios]
└─$ r2 -d mindreader2
WARN: Relocs has not been applied. Please use `-e bin.relocs.apply=true` or `-e bin.cache=true` next time
[0x7f1244cb3b00]> iz~py
31  0x00008ba0 0x00408ba0 9   10   .rodata ascii %s%c%s.py
33  0x00008bb3 0x00408bb3 12  13   .rodata ascii _pyi_main_co
67  0x00009128 0x00409128 15  16   .rodata ascii pyi-python-flag
69  0x00009148 0x00409148 18  19   .rodata ascii pyi-runtime-tmpdir
70  0x0000915b 0x0040915b 22  23   .rodata ascii pyi-contents-directory
71  0x00009172 0x00409172 29  30   .rodata ascii pyi-bootloader-ignore-signals
76  0x000091d8 0x004091d8 32  33   .rodata ascii Failed to copy file %s from %s!\n
82  0x0000930d 0x0040930d 4   5    .rodata ascii pyi-
179 0x00009f1b 0x00409f1b 16  17   .rodata ascii _pyinstaller_pyz
184 0x00009fe8 0x00409fe8 54  55   .rodata ascii Failed to pre-initialize embedded python interpreter!\n
185 0x0000a020 0x0040a020 67  68   .rodata ascii Failed to allocate PyConfig structure! Unsupported python version?\n
186 0x0000a068 0x0040a068 32  33   .rodata ascii Failed to set python home path!\n
189 0x0000a0e0 0x0040a0e0 45  46   .rodata ascii Failed to start embedded python interpreter!\n
288 0x0000ad98 0x0040ad98 40  41   .rodata ascii LOADER: failed to allocate pyi_argv: %s\n
```

#### Conclusión:

mindreader está compilado en `C` ya que tiene strings cortos, directos y muy legibles: nombres de frutas, mensajes simples, etc.

mindreader2 también está compilado en `C`, aunque el código esté programado en Pyhton por culpa del uso del loader `PyInstaller` que dentro tiene un intérprete de Pyhton embedido, esto es fácil de detectar ya que  tiene strings largos, muchos nombres técnicos e incluso el nombre directo del propio loader (pyi-, pyinstaller, embedded python interpreter, errores de PyInstaller, etc.), que hacen que la salida parezca "ruidosa" o "desordenada".

Aparte, mientras que mindreader pesa 14kb, mindreader2 pesa 7mb, otra demostración de que se está usando un loader de pyhton.

```
-rwxrwxr-x  1 kali kali   16464 May 18 17:59 mindreader
-rwxrwxr-x  1 kali kali 7441208 May 18 17:59 mindreader2
```

> Nota: al estar haciendo el ejercicio 5, me dí cuenta que si al ejecutar mindreader2 e interrumpes el programa con `Ctl + C` se puede ver por ahí que internamente el programa se llama mindreader.py
>```
>┌──(kali㉿kali)-[~/Desktop/reversing-binarios]
>└─$ ./mindreader2
>Password: ^CTraceback (most recent call last):
>  File "mindreader.py", line 4, in <module>
>KeyboardInterrupt
>[PYI-61157:ERROR] Failed to execute script 'mindreader' due to >unhandled exception!
>```

## Ejercicio 5

### - Ambos ejecutables tienen una flag, intenta hacer que el programa te de la flag. **NO** sirve encontrar la flag como un string, debes provocar que el ejecutable lo devuelva en tiempo de ejecución.

## Análisis del funcionamiento de `mindreader` y sus funciones

Accedemos al fichero `mindreader` y le hacemos el análisis automático

```
r2 mindreader
```

```
[0x00001140]> aaa
INFO: Analyze all flags starting with sym. and entry0 (aa)
INFO: Analyze imports (af@@@i)
INFO: Analyze entrypoint (af@ entry0)
INFO: Analyze symbols (af@@@s)
INFO: Recovering variables
INFO: Analyze all functions arguments/locals (afva@@@F)
INFO: Analyze function calls (aac)
INFO: Analyze len bytes of instructions for references (aar)
INFO: Finding and parsing C++ vtables (avrr)
INFO: Analyzing methods
INFO: Recovering local variables (afva)
INFO: Type matching analysis for all functions (aaft)
INFO: Propagate noreturn information (aanr)
INFO: Use -AA or aaaa to perform additional experimental analysis
```

Y procedemos a analizar todas las funciones con `alf`, podemos encontrar una función llamada `sym.adivinar_palabra` ubicada en la direción de memoria `0x00001229` 

```
[0x00001140]> afl
0x000010c0    1     10 sym.imp.puts
0x000010d0    1     10 sym.imp.__stack_chk_fail
0x000010e0    1     10 sym.imp.strcspn
0x000010f0    1     10 sym.imp.srand
0x00001100    1     10 sym.imp.fgets
0x00001110    1     10 sym.imp.strcmp
0x00001120    1     10 sym.imp.time
0x00001130    1     10 sym.imp.rand
0x00001140    1     37 entry0
0x00001170    4     34 sym.deregister_tm_clones
0x000011a0    4     51 sym.register_tm_clones
0x000011e0    5     54 sym.__do_global_dtors_aux
0x000010b0    1     10 fcn.000010b0
0x00001220    1      9 sym.frame_dummy
0x0000134c    1     13 sym._fini
0x000012d8    1    114 main
0x00001229    6    175 sym.adivinar_palabra      <----------
0x00001000    3     27 sym._init
```

### Función `main`:

Vamos a acceder a la función `sym.main` y a imprimirla para observar su comportamiento

```
[0x00001140]> s sym.main
[0x000012d8]> pdf
            ; DATA XREF from entry0 @ 0x1158(r)
┌ 114: int main (int argc, char **argv, char **envp);
│           ; var int64_t var_8h @ rbp-0x8
│           0x000012d8      f30f1efa       endbr64
│           0x000012dc      55             push rbp
│           0x000012dd      4889e5         mov rbp, rsp
│           0x000012e0      4883ec10       sub rsp, 0x10
│           0x000012e4      bf00000000     mov edi, 0                  ; time_t *timer
│           0x000012e9      e832feffff     call sym.imp.time           ; time_t time(time_t *timer)
│           0x000012ee      89c7           mov edi, eax                ; int seed
│           0x000012f0      e8fbfdffff     call sym.imp.srand          ; void srand(int seed)
│           0x000012f5      e836feffff     call sym.imp.rand           ; int rand(void)
│           0x000012fa      4863c8         movsxd rcx, eax
│           0x000012fd      48bacdcccc..   movabs rdx, 0xcccccccccccccccd
│           0x00001307      4889c8         mov rax, rcx
│           0x0000130a      48f7e2         mul rdx
│           0x0000130d      48c1ea03       shr rdx, 3
│           0x00001311      4889d0         mov rax, rdx
│           0x00001314      48c1e002       shl rax, 2
│           0x00001318      4801d0         add rax, rdx
│           0x0000131b      4801c0         add rax, rax
│           0x0000131e      4829c1         sub rcx, rax
│           0x00001321      4889ca         mov rdx, rcx
│           0x00001324      48c1e203       shl rdx, 3
│           0x00001328      488d05f12c..   lea rax, obj.palabras       ; 0x4020
│           0x0000132f      488b0402       mov rax, qword [rdx + rax]
│           0x00001333      488945f8       mov qword [var_8h], rax
│           0x00001337      488b45f8       mov rax, qword [var_8h]
│           0x0000133b      4889c7         mov rdi, rax                ; char *arg1
│           0x0000133e      e8e6feffff     call sym.adivinar_palabra
│           0x00001343      b800000000     mov eax, 0
│           0x00001348      c9             leave
└           0x00001349      c3             ret
```


1. Llama a `time(0)` y `srand()`: Esto se usa para inicializar el generador de números aleatorios con la hora actual.

2. Genera un número aleatorio con `rand()`, y hace una serie de operaciones matemáticas para elejir el índice de una palabra aleatoria del array `obj.palabras`

3. Carga esa palabra y la guarda en `rbp-0x8`.

4. Llama a `sym.adivinar_palabra(char *palabra)`, pasando como argumento la palabra seleccionada aleatoriamente.

5. Termina el programa. 

### Función `adivinar_palabra`:

Ahora nos moveremos a la función `sym.adivinar_palabra` y la imprimimos

```
[0x000012d8]> s sym.adivinar_palabra
[0x00001229]> pdf
            ; CALL XREF from main @ 0x133e(x)
┌ 175: sym.adivinar_palabra (char *arg1);
│           ; arg char *arg1 @ rdi
│           ; var int64_t canary @ rbp-0x8
│           ; var char *s1 @ rbp-0x70
│           ; var char *s2 @ rbp-0x78
│           0x00001229      f30f1efa       endbr64
│           0x0000122d      55             push rbp
│           0x0000122e      4889e5         mov rbp, rsp
│           0x00001231      4883c480       add rsp, 0xffffffffffffff80
│           0x00001235      48897d88       mov qword [s2], rdi         ; arg1
│           0x00001239      64488b0425..   mov rax, qword fs:[0x28]
│           0x00001242      488945f8       mov qword [canary], rax
│           0x00001246      31c0           xor eax, eax
│           0x00001248      488d05fb0d..   lea rax, str.Password:      ; 0x204a ; "Password: "
│           0x0000124f      4889c7         mov rdi, rax                ; const char *s
│           0x00001252      e869feffff     call sym.imp.puts           ; int puts(const char *s)
│           0x00001257      488b15122e..   mov rdx, qword [obj.stdin]  ; obj.__TMC_END__
│                                                                      ; [0x4070:8]=0 ; FILE *stream                                                         
│           0x0000125e      488d4590       lea rax, [s1]
│           0x00001262      be64000000     mov esi, 0x64               ; 'd' ; int size
│           0x00001267      4889c7         mov rdi, rax                ; char *s
│           0x0000126a      e891feffff     call sym.imp.fgets          ; char *fgets(char *s, int size, FILE *stream)
│           0x0000126f      488d4590       lea rax, [s1]
│           0x00001273      488d15db0d..   lea rdx, [0x00002055]       ; "\n"
│           0x0000127a      4889d6         mov rsi, rdx                ; const char *s2
│           0x0000127d      4889c7         mov rdi, rax                ; const char *s1
│           0x00001280      e85bfeffff     call sym.imp.strcspn        ; size_t strcspn(const char *s1, const char *s2)
│           0x00001285      c644059000     mov byte [rbp + rax - 0x70], 0
│           0x0000128a      488b5588       mov rdx, qword [s2]
│           0x0000128e      488d4590       lea rax, [s1]
│           0x00001292      4889d6         mov rsi, rdx                ; const char *s2
│           0x00001295      4889c7         mov rdi, rax                ; const char *s1
│           0x00001298      e873feffff     call sym.imp.strcmp         ; int strcmp(const char *s1, const char *s2)
│           0x0000129d      85c0           test eax, eax
│       ┌─< 0x0000129f      7511           jne 0x12b2
│       │   0x000012a1      488d05af0d..   lea rax, str.La_flag_es_NOTNULL ; 0x2057 ; "La flag es NOTNULL"
│       │   0x000012a8      4889c7         mov rdi, rax                ; const char *s
│       │   0x000012ab      e810feffff     call sym.imp.puts           ; int puts(const char *s)
│      ┌──< 0x000012b0      eb0f           jmp 0x12c1
│      ││   ; CODE XREF from sym.adivinar_palabra @ 0x129f(x)
│      │└─> 0x000012b2      488d05b10d..   lea rax, str.Acceso_denegado ; 0x206a ; "Acceso denegado"
│      │    0x000012b9      4889c7         mov rdi, rax                ; const char *s
│      │    0x000012bc      e8fffdffff     call sym.imp.puts           ; int puts(const char *s)
│      │    ; CODE XREF from sym.adivinar_palabra @ 0x12b0(x)
│      └──> 0x000012c1      90             nop
│           0x000012c2      488b45f8       mov rax, qword [canary]
│           0x000012c6      64482b0425..   sub rax, qword fs:[0x28]
│       ┌─< 0x000012cf      7405           je 0x12d6
│       │   0x000012d1      e8fafdffff     call sym.imp.__stack_chk_fail ; void __stack_chk_fail(void)
│       │   ; CODE XREF from sym.adivinar_palabra @ 0x12cf(x)
│       └─> 0x000012d6      c9             leave
└           0x000012d7      c3             ret
```


1. Imprime el mensaje `"Password:"` para pedir la contraseña al usuario.

2. Usa `fgets()` para leer hasta 100 caracteres de la entrada estándar y guarda el input en una variable local.

3. Usa `strcspn()` para buscar el salto de línea `'\n'` en la entrada y lo reemplaza por un terminador nulo `'\0'` para limpiar el input.

4. Compara el input limpio con la palabra secreta pasada como argumento usando `strcmp()`.

5. Si la comparación es igual (`strcmp` devuelve 0), imprime `"La flag es NOTNULL"`.

6. Si no coinciden, imprime `"Acceso denegado"`.

7. Realiza comprobación de stack canary para evitar corrupciones de pila y finaliza la función.


### Conclusión:

- La flag está codificada como uno de los elementos del array obj.palabras.

- Se elige una aleatoria en main.

- Se pasa a adivinar_palabra, y se compara con lo que el usuario escribe.

Por ende, podemos forzar la comparación para que siempre nos devuelva la flag.

## Modificando el comportamiento de `mindreader` con `radare2`

> Objetivo: Hacer que el programa siempre imprima `La flag es NOTNULL`, sin importar qué contraseña metas, modificando el binario para que el resultado de la comparación strcmp siempre sea “éxito” (es decir, strcmp == 0).

Accedemos al fichero `mindreader` en modo escritura

```
r2 -w mindreader
```

> (A partir de aquí nos estaremos fijando en el resultado de imprimir la función `adivinar_palabra`, detallado en la página anterior, para evitar redundancia de bloques de texto)

Nos movemos a la dirreción del salto condicional `jne`

```
[0x00004020]> s 0x129f
[0x0000129f]> 
```

Mostramos los bytes en hexadecimal indicando que queremos ver 4 bytes de esa dirección, puedo comprobar que `75` es el `opcode` (código de operación) de `jne` (jump if not equal). `11` es el desplazamiento: significa "salta 0x11 bytes hacia adelante".

```
[0x0000129f]> px 4
- offset -  9FA0 A1A2 A3A4 A5A6 A7A8 A9AA ABAC ADAE  F0123456789ABCDE
0x0000129f  7511 488d                                u.H.              
```

Seguidamente sobreescribimos el byte usando `wx` para escribir bytes hexadecimales en la posición actual, cambiando `75 (jne)` por `eb (jmp)`.

```
[0x0000129f]> wx eb
```

Comprobamos el cambio y nos salimos de `r2`

```
[0x0000129f]> px 4
- offset -  9FA0 A1A2 A3A4 A5A6 A7A8 A9AA ABAC ADAE  F0123456789ABCDE
0x0000129f  eb11 488d                                ..H.   
[0x0000129f]> q
```

Y...... no ha funcionado...

```
┌──(kali㉿kali)-[~/Desktop/reversing-binarios]
└─$ ./mindreader    
Password: 
hola
Acceso denegado
                                                                                                                                                             
┌──(kali㉿kali)-[~/Desktop/reversing-binarios]
└─$ ./mindreader
Password: 
pera
Acceso denegado
```

No nos desanimemos, vamos a ser más brutos, vamos a "destruir" la condicional por completo, volvemos a acceder en modo escritura y a la dirección de la condicional

```
r2 -w mindreader

[0x00004020]> s 0x129f
[0x0000129f]> 
```

Ahora vamos a utilizar `wa` para escribir instrucciones del lenguaje ensamblador en los dos primeros bytes, estableciendo `nop`, que significa "No Operation", es decir, no hacer nada. Primero lo escribimos en el primer byte

```
[0x0000129f]> wa nop
INFO: Written 1 byte(s) (nop) = wx 90 @ 0x0000129f

```
Nos movemos al segundo byte y volvemos a modificar la instrucción anterior
```
[0x0000129f]> s+ 1
[0x000012a0]> wa nop
INFO: Written 1 byte(s) (nop) = wx 90 @ 0x000012a0

```
Volvemos a la dirección inicial y comprobamos que se han modificado correctamente, siendo que `90` en hexadecimal se traduce a `nop`, por último nos salimos
```
[0x000012a0]> s 0x129f
[0x0000129f]> px 4
- offset -  9FA0 A1A2 A3A4 A5A6 A7A8 A9AA ABAC ADAE  F0123456789ABCDE
0x0000129f  9090 488d                                ..H.               
[0x0000129f]> q
```

Ahora sí, podemos comprobar que independientemente de la contraseña que introduzcamos siempre nos devuelve la flag

```
┌──(kali㉿kali)-[~/Desktop/reversing-binarios]
└─$ ./mindreader
Password: 
hola
La flag es NOTNULL
                                                                                                                                                             
┌──(kali㉿kali)-[~/Desktop/reversing-binarios]
└─$ ./mindreader  
Password: 
pera
La flag es NOTNULL
                                                                                                                                                             
┌──(kali㉿kali)-[~/Desktop/reversing-binarios]
└─$ ./mindreader
Password: 
lacabra
La flag es NOTNULL
```