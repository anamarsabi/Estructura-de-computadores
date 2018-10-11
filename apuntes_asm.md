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
		*ld: enlazador de GNU. ld combina un número de ficheros 			   objeto y archivos, reubica sus datos y enlaza referencias
       de  símbolos.  A  menudo el último paso en el proceso de construcción de un nuevo programa
       compilado para su ejecución es una llamada a ld.*

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



