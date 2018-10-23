# PRÁCTICAS ESTRUCTURA DE COMPUTADORES
# ENSAMBLADOR Básico

*http://ocw.uc3m.es/ingenieria-telematica/arquitectura-de-ordenadores/lecturas/html*

*Compilador* y *ensamblador* son términos distintos.
**Compilador** es el programa encargado de traducir de un lenguaje de programación a otro.
**Ensamblador** realiza la tarea del compilador pero especificamente la traducción es de lenguaje ensamblador a lenguaje máquina.
 El lenguaje máquina de un procesador es único e inmutable. Por lo tanto según la máquina puede ser distinto su lenguaje máquina de los de otras máquinas. 
En el caso concreto del sistema operativo Linux, se incluye como parte de las herramientas del sistema un compilador capaz de traducir de lenguaje ensamblador a lenguaje máquina cuyo nombre es **as**. Este programa lo suelen invocar compiladores como gcc ( C -> lenguaje máquina) 

## Programa en ensamblador
En un programa en ensamblador las palabras que comienzan por un punto se les denomina *directivas*.

### .data
Notifica al ensamblador que a continuación se encuentran definidos un conjunto de datos.

*.ascii*
Definición de uno o más strings entre comillas y separadas por comas. Cada símbolo de cada strings codifica con un byte en ASCII utilizando posiciones consecutivas de memoria. Se utilizan tantos bytes como la suma de los símbolos de cada string.

*.asciz* y *.string*
Similar a *.ascii* pero se diferencia porque codifica cada string añadiendo un byte con el valor cero al final del string. Este formato se utiliza para detectar el final del string. (Recordar cadenas de cararteres que acababan en \0).

*.byte*
Definición de valores numéricos almacenadoe en bytes. A esta directiva le sigue uno o varios valores separados por comas. Cuando el programa inicia la ejecución se han inicializado tantas posiciones en memoria como indica la directiva con los valores dados.

*.int* y *.long*
Definición de enteros de 32 bits. A esta directiva le sigue uno o varios numeros separados por comas. Se almacenan con 4 bytes almacenados en *little endian*.

*.word*
Definición de enteros de 16 bits.

*.quad*
Definición de enteros de 64 bits.

*.space <entero1> ,  <entero2>*
Reservar espacio en memoria.
<entero1> denota el número de bytes que se reservan.
<entero2> denota el valor que se utiliza para inicializar dichos bytes reservados. Si no se especifica este segundo entero, se inicializan a 0 por defecto.
Interés: reservar espacio que se debe inicializar al mismo valor o que será calculado y modificado por el propio programa.

![alt](https://upload.wikimedia.org/wikipedia/en/7/77/Big-little_endian.png)

### .section .text
Indica el cambio de sección. A partir de ahí comenzamos a emitir código. 

*.global _start*
Muestra punto de entrada a ld.
​		*ld: enlazador de GNU. ld combina un número de ficheros 			   objeto y archivos, reubica sus datos y enlaza referencias
​       de  símbolos.  A  menudo el último paso en el proceso de construcción de un nuevo programa
​       compilado para su ejecución es una llamada a ld.*

*_start:*
Punto de entrada ASM (como main en C)
*ASM: lenguaje ensamblador, es la abreviacion de assembly languaje)*


Todo programa ensamblador debe seguir el siguiente patrón:

`.data				#Comienzo del segmento de datos`

`<datos del programa>`


`.section .text		#Comienzo del código`

`.global _start`

`_start:				#Comienza "main"`

`<instrucciones>`


`ret					#obligatorio`

Una palabra seguida de **:** es la forma de definir una etiqueta o nombre de algo que luego se utilizará en el código del programa.
El compilador *gcc* asume que el punto de comienzo de programa está marcado por la presencia de  la etiqueta **main**. Por tanto, al escribir un programa que sea traducido por gcc se debe definir la etiqueta main en el lugar del código que contenga su primera instrucción máquina.

## Gestión de la pila
La pila se utiliza como  depósito temporal de datos del proggrama en ejecución.
Los programas en ensamblador tienen ciertas restricciones al manipular la pila.
La más importante de ellas es que la cima de la pila debe ser exactamente la misma antes del comienzo de la primera instrucción de un programa y antes de la instrucción `ret` que termina la ejecución del programa.
Todo programa ensamblador debe comenzar y terminar con isntrucciones de `push` y `pop` de los registros que se modifiquen en su interior porque por regla general, al finalizar la ejecución de un programa el valor de los registros de propósito general debe ser exactamente el mismo que tenían cuando se comenzó la ejecución(aunque en la práctica no todos los registros deben ser restaurados). El orden en el que se salvan y restauran los registros es el inverso debido a cómo se almacenan en la pila.


## Instrucciones de movimiento de datos

**MOV**

**`mov <source>, <dest>`**

`mov %regA, %regB` -> Mueve el contenido de %regA al registro %regB

`mov $inm, %reg` -> Mueve inm (valor numérico) al registro %reg

`mov %reg, mem` -> Mueve el contenido de %reg a la posición mem

`movs $inm, mem` -> Mueve inm(valor numérico codificado con los bits especificados por el sufijo s *(movs)*) al dato cuyo tamaño está especificado por el sufijo s y que está almacenado  en mem. 


**PUSH: Instrucción de carga sobre la pila**

**`push <algo`**

`push %reg` -> almacena el contenido de %reg en la posición anterior a la que apunta el puntero de la pila

`push mem` -> almacena el dato de 32 bits que está almacenado a partir de la posición mem en la posición anterior a la que apunta el puntero de pila

`push $inm` -> Almacena inm(valor numérico codificado con 32 bits) en la posición anterior a la que apunta el puntero de pila.

La instrucción `push` recibe un único operando y manipula siempre operandos de 32 bits, por lo tanto no se usan sufijos de tamaño. El procesador toma el valor del registro puntero de pila, le resta 4 y almacena el operando dado en los cuatro bytes de memoria a partir de la posición del puntero de pila.


**POP: Instrucción de descarga de la pila**

**`pop <algo>`**

`pop %reg` -> Almacena el contenido al que apunta el puntero de pila en %reg. Modifica el puntero a la cima para que apunte a la siguiente posición de la pila.

`pop mem` -> Almacena los 32 bits a los que apunta el puntero de pila a partir de la posición mem. Modifica el puntero a la cima para que apunte a la siguiente posición de la pila.

La instruccion `pop` recibe un único operando y manipula siempre operandos de 32 bits. El procesador toma el valor del registro puntero de pila y mueve ese dato al lugar que le indique el operando de la instrucción.  Tras esta transferencia, se suma el valor 4 al registro puntero de pila. 


**XCHG: Instrucción de intercambio**

**`xchg <registro> <registro>` o `xchg <registro> <dato>`** -> Intercambia los valores de sus operandos. Al menos uno de los operandos debe ser de tipo registro. Esta instrucción utiliza un reguistro temporal interno del procesador para intercambiar los operandos.

Ninguna de estas instrucciones de movimiento de datos modifica ninguno de los flags de la palabra de estado.



**LEAL**

**`leal <source>, <destination>, k`** -> Calcula expresiones aritmeticas de la forma x+k*y.  Donde  x=source , y =destination y k=1,2,4,8. Leal guarda en el registro destino el valor de la operación artmética sin meterse en memoria. Calcula direcciones sin hacer referencias a memoria. Se diferencia de mov en que esta instrucción, mov, almacena el contenido que hay en una dirección de memoria.




## Instrucciones aritméticas

**ADD**

**`add <sumando>, <sumando_destino>`** -> Recibe dos operando, los suma y los guarda en el segundo sumando(<sumando_destino>). Se machaca y se pierde el valor del segundo operando. 


**SUB**

**`sub <op1>, <op2>`** -> Resta <op2> - <op1> y el resultado se almacena en <op2>.


**INC** Y **DEC**

**`inc <algo>`** -> Esta instrucción recibe un único operando al que le suma el valor 1. Análogamente decrementando en uno el operando dado en el caso de **`dec <algo`**


**NEG**

**`neg <algo>`** -> Instrucción de cambio de signo, la operación que realiza es equivalente a multiplicar por -1. Se asume que el operando está codificado en complemento a 2. Esta instrucción asigna directamente el valor 1 al flag de acarreo.

**MUL**

**`mul <algo>`** -> Instrucción de multiplicación sin signo. Esta instrucción tiene dos operandos pero el segundo de ellos es implícito y dependiendo de los datos que maneje la operación serán: %al(8 bits = 1 Byte), %ax (2 bytes = 1 word) y %eax (4 bytes = Doubleword). Esta instrucción multiplica <algo>  por esos registros dependiendo del caso y almacena el resultado en %ax (si se multiplican dos números de 8 bits), %dx:%ax (si se multiplican dos números de 16 bits se almacena en el registro de 32bits resultante de concatenar los registros %dx y %ax, con %dx como parte más significativa), %edx:%eax (si se multiplican dos números de 32 bits se almacena en el registro de 64 bits resultante de concatenar estos dos registros con %edx como parte más significativa).

Si hay registros de 32 bits... ¿por qué se almacena el resultado de multiplicar dos números de 16 bits concatenando registros de 16 bits? Razones históricas, había procesadores anteriores que sólo tenían registros de 16 bits y para almacenar 32 bits había que hacer esta concatenación. Para mantener el lenguaje máquina se ha mantenido esto.


**DIV**

**`div <algo>`** -> Instrucción de división sin signo. Esta instrucción sólo especifica el divisor, el dividendo es implícito y tiene tamaño doble al del divisor. El dividendo se obiene de %ax, %dx:%ax ó %edx:%eax (16 bits, 32 bits o 64 bits respectivamente). La instrucción devuelve dos resultados: cociente y resto. El cociente se almacena en %al, %ax o %eax. El resto se almacena en %ah, %dx o %edx.


**IMUL**

Instrucción de multiplicación con signo
**`imul <algo>`** -> igual que mul.

**`imul <factor1>, <factor2>`** -> El segundo operador es el destino donde se guarda el resultado y además este <factor2> debe ser uno de los registros de propósito general. Los dos operandos son del mismo tamaño, luego el resultado también se guarda en un registro del mismo tamaño que los factores, con lo cual hay un mayor riesgo de overflow. 

**`imul <factor1>, <factor2>, <factor3>`** -> <factor1> debe ser una constante. <factor2> debe ser una posición de memoria o un registro. <factor3> es un registro de propósito general donde se guarda el resultado. En este caso pasa lo mismo con los tamaños que en el anterior. Como se solventa esto es realizando la multiplicación y obteniendo todos los bits del resultado y posteriormente los trunca para almacenar en destino.


**IDIV**

**`idiv <algo>`** -> Instrucción de división con signo. El comportamiento es igual que DIV (instrucción de división con signo) pero los factores son números enteros. 


## Instrucciones lógicas

**AND**,  **OR** Y **XOR**

**`and <algo1>, <algo2>`** -> Realiza la conjunción bit a bit del contenido de los operandos (recordamos que estos operandos pueden ser registros de los que se usará el contenido, constantes o posiciones de memoria). La conjunción bit a bit consiste en hacer la conjunción de los correspondientes bits de ambos operandos. Es necesario que ambos operandos tengan el mismo tamaño. Análogamente, con la disyunción bit a bit para el caso de **`or <algo1>, <algo2>`** y la disyunción exclusiva en **`xor <algo1>. <algo2>`**.


**NOT**

**`not <algo>`** -> Instrucción de negación: niega bit a bit el operando. No confundir con NEG. NOT es una instrucción lógica y NEG una instrucción aritmética que consiste en multiplicar por -1.


## Instrucciones de desplazamiento

**SAL/SAR**

**`sal <algo1>, <algo2>`** ó **`sar <algo1>, <algo2>`** -> Desplazamiento aritmético: desplaza el contenido del segundo operando a izquierda (sal, L de left) ó derecha (sar, R de right) tantas posiciones como indica el primer operando. En el caso de SAL, para cada desplazamient el bit más significativo se carga en el flag CF y el menos significativo se pone a 0. Para SAR esto es al contrario, el bit menos significativo se carga en el flag CF y el nuevo bit más significativo se pone al mismo valor del anterior (extensión de signo).  Esta instrucción no introduce ceros por la izquierda del operando, sino que replica el bit de mayor peso (bit de signo) en cada desplazamiento.
SAR sirve para dividir un operando entre una potencia entera de 2.


![alt](https://github.com/anamarsabi/Estructura-de-computadores/blob/master/images/sar.jpg?raw=true)

SAL sirve para multiplicar un operando, interpretado con signo, por una potencia de 2. Como esta instrucción equivale a multiplicar <algo2> * 2^<algo1>. El microprocesador detecta si se produce overflow al realizar el desplazamiento, registrándose este hecho en el bit OF del registro de estado. 

![alt](https://github.com/anamarsabi/Estructura-de-computadores/blob/master/images/shl.jpg?raw=true)


**SHL/SHR**

**`shl <algo1>, <algo2>`** ó **`shr <algo1>, <algo2>`** -> Desplazamiento lógico.
SHL es exactamente idéntico a SAL. Su objetivo es el mismo, multiplicar un operando por una potencia de 2. sal y shl son, de hecho, la misma instrucción y se codifican con el mismo código máquina.

![alt](https://github.com/anamarsabi/Estructura-de-computadores/blob/master/images/shl.jpg?raw=true)

SHR desplaza los bits del operando destino a la derecha tantos bits como indique el operando fuente. 

![alt](https://github.com/anamarsabi/Estructura-de-computadores/blob/master/images/shr.jpg?raw=true)


**RCL/RCR**

**`rcl <algo1>, <algo2>`** ó **`rcr <algo1>, <algo2>`** -> Instrucción de rotación con acarreo: rota el contenido del segundo operando concatenado con el flag CF a la izquierda (rcl) o a la derecha (rcr) tantas posiciones como indica el primer operando. 
RCL: para cada desplazamiento el bit más significativo se carga en el flag CF y éste pasa a ser el bit menos significativo.
RCR: para cada desplazamiento el bit menos significativo se carga en el flag CF y  éste pasa a ser el bit más significativo.
Los dos extremos del registro están unidos entre sí a través del flag de acarreo, el cual queda en medio de ellos. El bit que sale por un extremo va al flag del acarreo, y el bit original que estaba en el flag del acarreo entra al registro por el extremo opuesto.
Esta instrucción se utiliza para posicionar un determinado bit de un dato en el bit de acarreo y así poder consultar su valor. 
Si se fija el flag del acarreo de antemano, una rotación simple a través del acarreo puede simular un desplazamiento lógico o aritmético de una posición. Por esta razón, algunos microcontroladores solo tienen las funciones de rotar y rotar a través del acarreo y no se preocupan de tener instrucciones de desplazamiento aritmético o lógico.


![alt](https://github.com/anamarsabi/Estructura-de-computadores/blob/master/images/RCL.png?raw=true)

**ROR/ROL**

**`ror <algo1>, <algo2>`** ó **`rol <algo1>, <algo2>`** -> Instrucción de rotación sin acarreo: rota el contenido del segundo operando a la izquierda (rol) o a la derecha (ror) tantas posiciones como indica el primer operando. Si el desplazamiento es a la izquierda, para cada desplazamiento el bit más significativo pasa a ser el menos significativo. Si el desplazamiento es a la derecha, para cada desplazamiento el bit menos significativo pasa a ser el más significativo. Con estas instrucciones se consigue una estructura circular como si los extremos izquierdo y derecho del registro estuvieran conectados. Esta instrucción es frecuentemente usada en criptografía digital.

![alt](https://github.com/anamarsabi/Estructura-de-computadores/blob/master/images/ROL.png?raw=true)


## Instrucciones de salto

**JMP**

**`jmp <algo>`** -> Instrucción de salto condicional: cambia la secuencia de ejecución del procesador, éste pasa ejecutar a continuación la instrucción almacenada a partir de la posición de memoria en el operando. 
Hay más instrucciones de salto condicional específicas

**CALL**

**`call <algo`** -> Instrucción de llamada a subrutina: invoca a la subrutina cuya primera instrucción está en la posición de memoria que indica el operando.  Deposita en la cima de la pila la dirección de retorno, es decir, la dirección de la instrucción que sigue a esta en el flujo de ejecución.

**RET**

**`ret `** -> Instrucción de retorno de subrutina: retorna la ejecución a la instrucción cuya dirección está almacenada en la cima de la pila. Esta dirección se saca de la pila. 


## Instrucciones de comparación y comprobación

**CMP**

**`cmp <algo1>, <algo2>`** -> Instrucción de comparación. Resta el primer operando al segundo y modifica los flags con el resultado, el cual no se almacena en ningún lugar. Estos flags que han sido modificados se pueden usar para cambiar el flujo de ejecución mediante una isntrucción de salto condicional.

**TEST**

**`test <algo1>, <algo2>`** -> Instrucción de comprobación.  Realiza la conjunción bit a bit de los operandos y modifica los flags con el resultado, el cual tampoco que almacena en ningún lugar.  Muchas veces se hace un salto condicional después.




## Registros enteros 

Los registros son circuitos digitales internos del procesador que se comportan igual que las celdas de memoria, es decir, permiten las operaciones de lectura y escritura de datos pero a una velocidad mucho mayor, pues no requieren la comunicación con ningún circuito externo al procesador. Los registros que ofrece un procesador se identifican por su nombre y son susceptibles de ser utilizados al escribir programas en ensamblador. 

La arquitectura IA-32 ofrece 16 registros básicos para la ejecución de programas:
6 registros de segmento
Un registro de estado y control
Un registro contador del programa
8 registros de propósito general

**Registros de propósito general**

 %eax, %ecx, %edx, %esi, %edi, %ebp y %esp. 

Todos ellos tiene un tamaño de 32 bits y su principal cometido es almacenar datos temporales necesarios para la ejecución de programas. En estos registrso se guardan temporalmente aquellos datos que necesita el procesador más a menudo, de esta forma se obtiene un mejor rendimiento en la ejecución. Por ejemplo, si un dato se utiliza varias veces seguidas, en lugar de llerlo de memoria cada vez es mejor almacenarlo al principio en uno de los registros de propósito general y referirse a esa copia cada vez que sea necesario. El procesador permite referirse a ciertas porciones de los registros de propósito general con nombres diferentes. Permite manipular los 16 bits de menos peso suprimiendo la "e" del comienzo del nombre del registro. Para los registros %eax, %ebx, %ecx, %edx se permite manipular los dos bytes de menos peso de forma independiente suprimiendo la "e" y sustituyendo la "x" por "h" para el byte de más peso o "l" para el de menos peso. 

![alt](https://github.com/anamarsabi/Estructura-de-computadores/blob/master/images/Selecci%C3%B3n_041.png?raw=true)

Atención! Los registros %esp y %ebp son los que trabajan con la pila, luego mejor no modificarlos manualmente.

Pasar los argumentos a través de pila permite hacer recursividad. 





**Registro de estado y control**

El registro de estado y control es el registro de estado en los microprocesadores Intel x86 que contiene el estado actual del procesador. Este registro es de 16 bits de ancho. Sus sucesores son los registros EFLAGS Y RFLAGS de 32 bits y 64 bits respectivamente.  Por lo tanto, el registro de estado y control de la arquitectura de IA-32 es EFLAGS. De los 32 bits tan solo 18 de llos contienen información sobre el estado y control, el resto contienen un valor fijo.
Durante la ejecución de isntrucciones existen situaciones especiales que convienen ser reflejadas en un registro para su posible consulta.  Aparte de las condiciones de funcionamiento, existe un conjunto de funcionalidades que es preciso activar o desactivar en ciertos momentos de la ejecución de un procesador. Por ejemplo, la arquitectura IA-32 permite que una isntrucción sea interrumpida y se pase a ejecutar momentáneamente un conjunto de instrucciones , mediante un bit de control de permite o prohibe que estas interrupciones se produzcan.
Las condiciones que representan los bits más importantes de este registro son:

*CF -> Bit de acarreo*: Su valor es 1 si una operación aritmética con naturales ha producido acarreo. Se utiliza para detectar situaciones de desbordamiento.

*PF -> Bit de paridad*:  Su valor es 1 si el byte menos significativo de una operación aritmética contiene un número impar de unos.

*AF -> Bit de ajuste*: Su valor es 1 si se produce acarreo : en operaciones aritméticas en la codificación BCD(binary coded decimal: representar números decimales en el sistema binario donde cada dígito decimal es codificado con una secuencia de 4 bits).

*ZF -> Bit de cero*: Su valor es 1 si ele resultado de la última operación aritmética ha sido cero.

*SF -> Bit de signo*: su valor es idéntico al bit más significativo del resultado que corresponde con el bit de signo, cero si es positivo y 1 si es negativo.

*OF -> Bit de desbordamiento*: Su valor es 1 si el entero obtenido como resultado no puede ser rrepresentado en complemento a 2 con el número de bits utilizado.

El valor de los bits de estado fluctúa continuamente dependiendo de los resultados aritméticos producidos a lo largo de la ejecución del programa. El valor de estos bits modifican el comportamiento de un subconjunto muy relevante de instrucciones del procesador, entre ellas los saltos condicionales. 

