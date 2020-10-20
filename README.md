In deze oefenzitting leren jullie het besturingssysteem xv6 kennen.

- [Voorbereiding](#voorbereiding)
- [GitHub classroom](#github-classroom)
  - [Persoonlijke repository](#persoonlijke-repository)
  - [Owner file](#owner-file)
- [xv6 shell](#xv6-shell)
- [User space programma bekijken](#user-space-programma-bekijken)
  - [Libraries](#libraries)
  - [Programma afsluiten](#programma-afsluiten)
- [Eigen user space programma toevoegen](#eigen-user-space-programma-toevoegen)
- [Zelfreflecterend proces](#zelfreflecterend-proces)
- [Communicerende processen](#communicerende-processen)
  - [Memory layout opstellen](#memory-layout-opstellen)
  - [Kopie van het proces maken](#kopie-van-het-proces-maken)
  - [Waarden doorsturen van child naar parent](#waarden-doorsturen-van-child-naar-parent)
  - [Waarden printen](#waarden-printen)
  - [Indienen](#indienen)

# Voorbereiding

Ter voorbereiding van deze oefenzitting wordt je verwacht:
* Een werkende Linux-omgeving te hebben. Volg stap 1 van [deze tutorial](https://github.com/informaticawerktuigen/klaarzetten-werkomgeving) voor meer uitleg.
* Een werkende versie van xv6-riscv te hebben. Volg [deze tutorial](https://github.com/besturingssystemen/klaarzetten-werkomgeving) voor meer uitleg.
* Hoofdstuk 1 van het [xv6 boek](https://github.com/mit-pdos/xv6-riscv-book/) gelezen te hebben.

# GitHub classroom

## Persoonlijke repository

De submissie van de permanente evaluatie zal gebeuren via GitHub classroom. 
Jullie krijgen hiervoor een kopie van de xv6 repository.
Hierin kunnen jullie met `git` zelf wijzigingen toevoegen en committen.

* Klik op [deze link](https://classroom.github.com/a/uaBpI1Zz) om een persoonlijke repository aan te maken.

Wanneer je een e-mail krijg van GitHub dat je repository klaar is, moet je deze clonen naar je eigen machine. Dit kan enkele minuten duren.

* Clone je persoonlijke repository

```shell
git clone https://github.com/besturingssystemen/xv6-permanente-evaluatie-<GitHubUsername>.git
```

* Verifieer dat je repository correct gecloned is door `make qemu` uit te voeren.

```shell
cd xv6-permanente-evaluatie-<GitHubUsername>
make qemu
```

Indien `make qemu` ervoor zorgt dat xv6 opstart, is je repository correct gecloned.

## Owner file

Om ervoor te zorgen dat wij weten welke student hoort bij een GitHub account vragen we jullie een bestand `OWNER` te committen met daarin je naam.

* Maak een bestand met je naam in
```shell
cd xv6-permanente-evaluatie-<GitHubUsername>
echo "Voornaam Achternaam" > OWNER
```
* Indien je `git` voor het eerst gebruikt in je Linux-installatie moet je het configureren met 
```shell 
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com 
```

* Commit en push vervolgens het `OWNER` bestand met `git`
```shell
git add OWNER
git commit -m "Added OWNER file"
git push origin master
```` 

* Verifieer dat je OWNER bestand correct toegevoegd is aan je remote repository door naar de web interface van je repository te surfen
```shell
firefox https://github.com/besturingssystemen/xv6-permanente-evaluatie-GithubUsername
```

# xv6 shell

Wanneer je xv6 start met ```make qemu``` beland je in een simpele shell-omgeving.

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
* Bekijk de code van het programma `cat`.
  ```shell
  gedit xv6-riscv/user/cat.c
  ```

## Libraries
De libraries die je kan terugvinden in de `#include`-statements zijn niet de standaard libraries die we kennen wanneer we C programmeren.

> :information_source: De [C Standard Library](https://en.wikipedia.org/wiki/C_standard_library) bestaat uit een verzameling nuttige functies die door C-programma's gebruikt kunnen worden. De implementatie van deze functies hangt echter af van het onderliggende besturingssysteem. Op Ubuntu en vele andere Unix-distributies heet deze implementatie `glibc` of [The GNU C Library](https://www.gnu.org/software/libc/). Een andere populaire implementatie heet [`musl`](https://musl.libc.org/).

xv6 biedt maar een zeer beperkte implementatie van libc aan. Daardoor kunnen we niet zomaar `#include <stdio.h>` gebruiken om bijvoorbeeld de functie `printf` op te roepen. 
De libc-functies die xv6 aanbiedt kan je terugvinden in `xv6-riscv/user/user.h`.

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

Merk op dat in xv6 een user space programma afgesloten wordt door de oproep ```exit(0);``` in plaats van te returnen uit main. Dit is het gevolg van het feit dat xv6 geen standaard C runtime implementeert. Een C runtime zoals [crt0](https://en.wikipedia.org/wiki/Crt0) is typisch verantwoordelijk voor het oproepen van de main-functie en na een return uit main het proces correct af te sluiten. Kijk [hier](https://stackoverflow.com/questions/3463551/what-is-the-difference-between-exit-and-return) voor meer informatie.

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

Gebruik onderstaande `struct memlayout` om deze waarden te bewaren.



```c
struct memlayout {
    void* text;
    void* data;
    void* stack;
    void* heap;
};
```


Onderstaande methoden kan je gebruiken om de adressen van elke sectie te vinden:

* Zoek een adres op de stack door een lokale variabele te declareren in een functie. Deze variabele zal altijd gealloceerd worden op de call stack van je proces. Het adres van deze variabele is dus een adres op de stack. 
* Zoek een adres op de heap door `sbrk(1)` op te roepen. De return-waarde van deze system call geeft de oude waarde terug van de [`program break` (meer info)](https://stackoverflow.com/questions/6338162/what-is-program-break-where-does-it-start-from-0x00).
* Zoek een adres in de .text section door het adres van een functie te printen. Het adres van een functie kan je verkrijgen door de naam van de functie te typen (zonder `()`)
    ```c
    printf("Address of func: %p", (void*) func)
    ```
* Zoek een adres in de .data section door een globale variabele te declareren en het adres van deze variabele op te vragen

Zorg ervoor dat de main-functie van je programma een `struct memlayout` aanmaakt. Sla in elk veld van de struct een correct adres op uit de bijhorende sectie van het programma.

Voeg vervolgens de onderstaande snippet toe aan je code. 
`get_values_from_layout` maakt een kopie van de waarden die in de adressen van `struct memlayout` opgeslagen zijn.

```c
struct memvalues {
    int data;
    int stack;
    int heap;
};

struct memvalues get_values_from_layout(struct memlayout layout){
    struct memvalues result;
    result.data = *(int*) layout.data;
    result.stack = *(int*) layout.stack;
    result.heap = *(int*) layout.heap;
    return result;
}

void print_memory(struct memlayout layout, struct memvalues values){
    printf("%p (.text)\n%p:%d (.data)\n%p:%d (.stack)\n%p:%d (.heap)\n",
            layout.text,
            layout.data,
            values.data,
            layout.stack,
            values.stack,
            layout.heap,
            values.heap);
}
```

* Gebruik de functie `get_values_from_layout` om een `struct memvalues` te verkrijgen vanuit je memory layout.
* Print ten slotte alle waarden door aan `print_memory` de `struct memlayout` en de `struct memvalues` door te geven.

De output van je programma zou er als volgt kunnen uitzien:
```shell
$ introspection
0x0000000000000044 (.text)
0x0000000000000E34:42 (.data)
0x0000000000002FCC:42 (.stack)
0x0000000000003000:0 (.heap)
```
> :exclamation: De bovenstaande werkwijzen geven telkens een adres terug uit een bepaalde sectie van het programma. 
> Deze adressen zijn niet noodzakelijk de start- of eindadressen van deze secties.
> Ze vallen wel telkens in de range [`section start addr`, `section end addr`]



# Communicerende processen

We weten nu hoe we user space programma's kunnen toevoegen aan xv6. Als laatste deel van deze oefenzitting en als *permanente evaluatie* is het de bedoeling dat je het programma `introspection.c` uitbreidt.

Zorg ervoor dat je programma onderstaande stappen uitvoert:

## Memory layout opstellen
  1. Maak een correcte `struct memlayout` aan zoals in het vorige deel
  2. Schrijf de waarde `42` naar alle addressen in de `struct memlayout` (behalve het text-adres). Je kan hiervoor onderstaande functie gebruiken.

```c
void write_value_to_layout(int value, struct memlayout* layout){
    *((int*) layout->data) = value;
    *((int*) layout->stack) = value;
    *((int*) layout->heap) = value;
}
```

## Kopie van het proces maken 

  3. Open een `pipe`
  4. Maak een `child` proces aan met behulp van `fork()`
  5. Schrijf de waarde `7` naar alle addressen in de `struct memlayout` (behalve het text-adres). Doe dit enkel in het `child`-proces.
  6. Maak in het `child`-proces een `struct memvalues` met de functie `get_values_from_layout`

## Waarden doorsturen van child naar parent
  7. Stuur deze `struct memvalues` door naar de `parent` met behulp van de pipe

```C
//we assume the write-end of the pipe in pipes_fd[1] and the memory values in memvals
write(pipes_fd[1], memvals, sizeof(struct memvalues))    
```
8. Lees in het `parent`-proces de `memvalues` van het `child` proces uit de pipe. Bewaar het resultaat in een nieuwe `struct memvalues`.

## Waarden printen

9.  Gebruik `print_memlayout` in het `parent`-proces om de `memvalues` van het `child`-proces uit te printen
10. Vraag in het `parent`-proces de eigen `memvalues` op met behulp van `get_values_from_layout` 
11. Gebruik `print_memlayout` in het `parent`-proces om de eigen `memvalues` uit te printen

> :bulb: In hoofdstuk 1 van het xv6 boek kan je codevoorbeelden met verklaring vinden die gebruik maken van `fork` en `pipe`

## Indienen

Dit deel van de opgave moet ingediend worden en telt mee voor de permanente evaluatie van de oefeningen.
* Commit en push het bestand `introspection.c` naar je repository
```shell
git add introspection.c
git commit -m "Added introspection program"
git push origin master
```

> :bulb: Controleer op de webpagina van je repository of het bestand correct gecommit is.