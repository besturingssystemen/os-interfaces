In deze oefenzitting leren jullie het besturingssysteem xv6 kennen.

- [Voorbereiding](#voorbereiding)
- [xv6 shell](#xv6-shell)
- [User space programma bekijken](#user-space-programma-bekijken)
  - [Libraries](#libraries)
  - [Programma afsluiten](#programma-afsluiten)
- [Eigen user space programma toevoegen](#eigen-user-space-programma-toevoegen)
- [Zelfreflecterend proces](#zelfreflecterend-proces)
- [Communicerende processen](#communicerende-processen)

# Voorbereiding

Ter voorbereiding van deze oefenzitting wordt je verwacht:
* Een werkende Linux-omgeving te hebben. Volg stap 1 van [deze tutorial](https://github.com/informaticawerktuigen/klaarzetten-werkomgeving) voor meer uitleg.
* Een werkende versie van xv6-riscv te hebben. Volg [deze tutorial](https://github.com/besturingssystemen/klaarzetten-werkomgeving) voor meer uitleg.
* Hoofdstuk 1 van het [xv6 boek](https://github.com/mit-pdos/xv6-riscv-book/) gelezen te hebben.

# xv6 shell

Wanneer je xv6 start beland je in een simpele shell-omgeving.

* Voer het commando ``ls`` uit in de xv6 shell
    ```shell
    ls
    ```
Het resultaat van het ls-commando toont de root directory van het file system van xv6. In de startdirectory staan alle user space programma's. Deze programma's kan je uitvoeren vanuit de xv6 shell.
Daarnaast staat er ook een README-bestand.

* Lees `README` met behulp van het `cat`-commando

    ```shell
    cat README
    ```

* Maak een folder aan genaamd `testfolder` en `cd` naar die folder.
  
  ```shell
  mkdir testfolder
  cd testfolder
  ```
* Verifieer nu met `ls` dat je in de lege folder zit
    ```shell
    ls
    ```

Je krijgt nu de melding `exec ls failed`. De xv6 shell is namelijk een zeer simpele shell. 

Wanneer je in `bash` (de standaard shell in de meeste Linux-distributies) een commando uitvoert, zoekt `bash` in alle directories in een variabele genaamd `$PATH` naar dit programma.
De shell van xv6 (het programma `sh`) heeft geen `$PATH`-variabele en zoekt dus enkel in de huidige directory naar uitvoerbare programma's. 
Je kan alsnog `ls` uitvoeren door een relatief of absoluut pad te specifiëren.

* Voer ls uit in `sh` met een relatief pad

    ```shell
    ../ls
    ```
* Voer ls uit in `sh` met een absoluut pad 
    ```shell
    /ls
    ```


# User space programma bekijken

De broncode van de user space programma's die we net hebben uitgevoerd staat in de folder `xv6-riscv/user` in je Git-repository.

* Sluit de xv6-omgeving met <kbd>CTRL</kbd>+<kbd>A</kbd> <kbd>x</kbd>.
* Bekijk de code van het programma `ls`.
  ```shell
  gedit xv6-riscv/user/ls.c
  ```

## Libraries
De libraries die je kan terugvinden in de `#include`-statements zijn niet de standaard libraries die we kennen wanneer we C programmeren.

> :information_source: De [C Standard Library](https://en.wikipedia.org/wiki/C_standard_library) bestaat uit een verzameling nuttige functies die door C-programma's gebruikt kunnen worden. De implementatie van deze functies hangt echter af van het onderliggende besturingssysteem. Op Ubuntu en vele andere Unix-distributies heet deze implementatie `glibc` of [The GNU C Library](https://www.gnu.org/software/libc/). Een andere populaire implementatie heet [`musl`](https://musl.libc.org/).

xv6 biedt geen implementatie van libc aan. Daardoor kunnen we niet zomaar `#include <stdio.h>` gebruiken om bijvoorbeeld de functie `printf` op te roepen. 
Als alternatief biedt xv6 enkele eigen libc-functies
aan.

* Open het bestand `xv6-riscv/user/user.h`

    ```shell
    gedit xv6-riscv/user/user.h
    ```

De functies in dit bestand kunnen gebruikt worden door alle user space programma's. In plaats van de standaard C header files te includen (zoals `stdio.h`) zullen we in xv6 telkens het bestand `user/user.h` includen. 

De functies in `user/user.h` maken zelf gebruik van types gedefinieerd in `kernel/types.h`. Start dus standaard een user space programma voor xv6 met

```c
#include "kernel/types.h"
#include "user/user.h"
```
## Programma afsluiten

Merk op dat in xv6 een user space programma afgesloten wordt door de oproep ```exit(0);``` in plaats van te returnen uit main. Ook dit is het gevolg van het feit dat xv6 geen standaard C library aanbiedt. Kijk [hier](https://stackoverflow.com/questions/3463551/what-is-the-difference-between-exit-and-return) voor meer informatie.

# Eigen user space programma toevoegen

We zijn klaar om een simpel user space programma toe te voegen aan xv6.

* Maak een bestand `helloworld.c` in de directory `xv6-riscv/user`

```shell
cd xv6-riscv
touch user/helloworld.c
```

* Schrijf in dit bestand een simpel C-programma dat de string *Hello, world!* print naar de terminal.

```shell
gedit user/helloworld.c &
```

Bij het compileren van het besturingssysteem met `make qemu` worden ook alle programma's in de directory `xv6-riscv/user` gecompileerd naar Risc-V. Dit wordt gespecifieerd door middel van de `Makefile` in `xv6-riscv`.  

We voegen nu het helloworld-programma toe aan de Makefile.


* Open het bestand `xv6-riscv/Makefile`
```shell
gedit Makefile &
```
* Zoek naar de definitie van UPROGS en voeg het programma toe
```
UPROGS=\
    $U/_cat\
    ...
    $U/_helloworld\
    ...
    $U/_floats\
```
* Compileer `xv6` en start via qemu
```shell
make qemu
```
* Voer het programma uit
```shell
$ helloworld
Hello, World!
```

# Zelfreflecterend proces

Voeg nu zelf een userspace programma toe genaamd `introspection.c`.
De bedoeling is dat dit programma zijn eigen memory layout uitprint.

We zijn geïnteresseerd in de locatie van:
* de stack
* de heap
* de .text sectie (de code van je programma)
* de .data sectie (de global variables van je programma)



Onderstaande methoden kan je gebruiken om de adressen in deze secties te zoeken:

* Zoek een adres op de stack door gebruik te maken van [deze truc](https://stackoverflow.com/questions/20059673/print-out-value-of-stack-pointer)
    ```c
    void print_stack_address() {
        void* p = 0;
        printf("%p", (void*)&p);
    }
    ```

Deze code maakt een lokale variabele aan en print vervolgens het adres van die variabele. Lokale variabelen worden gealloceerd in de call stack van een proces. De locatie van een lokale variabele is dus altijd een adres in de huidige stack.

* Zoek een adres op de heap door `sbrk(1)` op te roepen. De return-waarde van deze system call geeft de oude waarde terug van de [`program break` (meer info)](https://stackoverflow.com/questions/6338162/what-is-program-break-where-does-it-start-from-0x00).
* Zoek een adres in de .text section door het adres van een functie te printen. Het adres van een functie kan je verkrijgen door de naam van de functie te typen (zonder `()`)
    ```c
    printf("Address of func: %p", (void*) func)
    ```
* Zoek een adres in de .data section door een global variable te declareren en het adres van deze variabele op te vragen
    ```c
    int global = 0;
    
    int func(){
        printf("Address of global: %p", &global);
    }
    ```
De output van je programma zou er als volgt kunnen uitzien
```shell
$ introspection
0x0000000000002FC0 is an address in my stack
0x0000000000003000 is an address in my heap
0x0000000000000010 is an address in my text section
0x0000000000000E2C is an address in my data section
```


# Communicerende processen

We weten nu hoe we user space programma's kunnen toevoegen aan xv6. Als laatste deel van deze oefenzitting is het de bedoeling dat je het programma `introspection.c` aanpast.

Zorg ervoor dat:
  * Het programma twee verschillende processen aanmaakt met behulp van `fork()`
  * Het child-proces de waarde van zijn stack en heap pointers doorstuurt naar het parent proces via een pipe
  * Het parent proces print vervolgens de waarde van zijn eigen stack- en heap pointer, en die van zijn child.

Deze oefening telt voor permanente evaluatie.

