# Memory Management

Sa zicem ca avem:

```python
student_name = 'Matei'
```

In spate se intampla urmatorul lucru:

-   Pe heap: este alocata memorie pentru stringul 'Matei'

-   Pe stack: este alocata memorie pentru referinta catre stringul 'Matei', deci cumva student_name contine o referinta (pointerul) catre o zona de memorie de pe Heap

### Mutable vs Imutable

În limbaje ca Java sau C++, există tipuri primitive (care stau direct pe stack).

-   În Python, nu există primitive. Până și un simplu întreg (x = 5) este un obiect complet pe Heap, care conține mai mult decât valoarea 5 (conține tipul, referința counts, etc.).

-   Immutable: String-urile, Tuplele și Integers. Dacă încerci să modifici un string, Python creează de fapt un obiect nou în memorie.

-   Fiecare obiect are un Reference Count. Daca RC = 0, adica nicio variabila nu ii da reference atunci  Garbage Collectorul isi va face treaba si ii da dealocate la memorie

-   Generational Garbage Collector: Python are și un al doilea mecanism pentru "Cyclic References" (când Obiectul A trimite la B, și B trimite la A). RC nu le poate șterge pe acestea, așa că GC-ul intervine periodic să curețe aceste "insule" izolate de memorie.

### Interpreter

-   Call Stack-ul nu doar "ține minte unde ești", ci stochează Stack Frames. Fiecare funcție apelată primește un "cadru" care conține variabilele locale și argumentele acelei funcții. Când funcția se termină, cadrul este șters (LIFO - Last In, First Out).

The python interpreter executes you code line by line

-   Instruction memory -> memory that holds your code instructions in bytecode form. The Python Interpreter reads your code from here

-   Heap Memory -> a giant pile of memory that Python grabs chunks from for storing data

-   Call Stack -> memory that Python uses to keep track of where your code it is

### Memory

Data -> represented as binary digits in memory:

-   Bits are grouped together into groups of 8, called bytes

-   Python uses groups of bytes to represent data such as integers, floats, strings, lists, etc.

-   Every byte of memory has a unique memory address