# Hoja de referencia de Go / Go Cheat Sheet

Traducido de [Golang cheat sheet de Ariel Mashraki](https://github.com/a8m/go-lang-cheat-sheet)\
Las *keywords* en su mayoría se conservan en su idioma original.

# Índice
1. [Sintaxis básica](#sintaxis-básica)
2. [Operadores](#operadores)
  * [Aritméticos](#aritméticos)
  * [Comparación](#comparación)
  * [Lógicos](#lógicos)
  * [Otros](#otros)
3. [Declaraciones](#declaraciones)
4. [Funciones](#funcioes)
  * [Funciones como valores y closures](#funciones-como-valores-y-closures)
  * [Funciones variádicas](#funciones-variádicas)
5. [Tipos incorporados](#tipos-incorporados)
6. [Conversión de tipos](#conversión-de-tipos)
7. [Paquetes](#paquetes)
8. [Estructuras de control](#estructuras-de-control)
  * [If](#if)
  * [Bucles](#bucles)
  * [Switch](#switch)
9. [Arrays, Slices, Ranges](#arrays-slices-ranges)
  * [Arrays](#arrays)
  * [Slices](#slices)
  * [Operaciones con Arrays y Slices](#operaciones-con-arrays-y-slices)
10. [Maps](#maps)
11. [Structs](#structs)
12. [Pointers](#pointers)
13. [Interfaces](#interfaces)
14. [Incrustación](#incrustación)
15. [Errores](#errores)
16. [Concurrencia](#concurrencia)
  * [Goroutines](#goroutines)
  * [Channels](#channels)
  * [Channel Axioms](#channel-axioms)
17. [Impresión](#impresión)
18. [Snippets](#snippets)
  * [Http-Server](#http-server)

## Créditos

La mayoría del código ejemplo de [A Tour of Go](http://tour.golang.org/), el cual es una excelente introdución a Go.
Si eres nuevo en Go, toma ese tour. En serio.

## Go en resumen

* Lenguaje imperativo
* Tipado estático
* Sintaxis similar a la de C (pero con menos paréntesis y punto y coma) y la estructura a la de Oberon-2
* Compila a código nativo (nada de JVM)
* No tiene clases, pero sí *structs* con métodos
* Interfases
* No hay implementación de herencias. Pero existen los [tipos embebidos](http://golang.org/doc/effective%5Fgo.html#embedding).
* Las funciones son ciudadanos de primera clase
* Las funciones pueden retornar valores múltiples
* Tiene closures
* Pointers, pero no pointer arithmetic
* Concurrencia incorporada: Goroutines y Channels

# Sintaxis básica

## Hola Mundo
Archivo `hola.go`:
```go
package main

import "fmt"

func main() {
    fmt.Println("Hola mundo")
}
```
`$ go run hola.go`

## Operadores
### Aritméticos
|Operador|Descripción|
|--------|-----------|
|`+`|suma|
|`-`|resta|
|`*`|multiplicación|
|`/`|división|
|`%`|remanente|
|`&`|y a nivel de bit|
|`|`|o a nivel de bit|
|`^`|xor a nivel de bit|
|`&^`|bit clear (y no)|
|`<<`|shift izquierdo|
|`>>`|shift derecho|

### Comparación
|Operador|Descripción|
|--------|-----------|
|`==`|igual|
|`!=`|no igual|
|`<`|menor que|
|`<=`|igual o menor que|
|`>`|mayor que|
|`>=`|igual o mayor que|

### Lógicos
|Operador|Descripción|
|--------|-----------|
|`&&`|y lógico|
|`||`|o lógico|
|`!`|no lógico|

### Otros
|Operador|Descripción|
|--------|-----------|
|`*`|dirección de / crear pointer|
|`&`|desreferenciar pointer|
|`<-`|enviar / recibir (ver 'Channels' abajo)|

## Declaraciones
El tipo (*type*) va después del identificador!
```go
var foo int // declaración sin inicialización
var foo int = 42 // declaración con inicialización
var foo, bar int = 42, 1302 // declarar e inicializar múltiples variables a la vez
var foo = 42 // tipo omitido, será inferido
foo := 42 // anotación corta, sólo dentro de funciones, omite la keyword var, el tipo siempre está implícito
const constante = "Esta es una constante"
```

## Funciones
```go
// una función simple
func nombreDeLaFuncion() {}

// función con parámetros (nuevamente, los tipos después de los identificadores)
func nombreDeLaFuncion(parametro1 string, parametro2 int) {}

// múltiples parámetros del mismo tipo
func nombreDeLaFuncion(parametro1, parametro2 int) {}

// declaración del tipo de retorno
func nombreDeLaFuncion() int {
    return 42
}

// Puede retornar valores múltiples a la vez
func retornoMultiple() (int, string) {
    return 42, "foobar"
}
var x, str = retornoMultiple()

// Retorno múltiple nombrado
func retornoMultiple2() (n int, s string) {
    n = 42
    s = "foobar"
    // n y s serán retornados
    return
}
var x, str = retornoMultiple2()

```

### Funciones como Valores y Closures
```go
func main() {
    // asigna una función a un nombre
    suma := func(a, b int) int {
        return a + b
    }
    // use the name to call the function
    fmt.Println(suma(3, 4))
}

// Closures, ámbito léxico: Las funciones pueden acceder a valores que
// estuvieron a su alcance (scope) al momento de definir esa función
func scope() func() int{
    variablexterna := 2
    foo := func() int { return variablexterna}
    return foo
}

func otro_scope() func() int{
    // no compilará porque variablexterna y foo no fueron definidos en su alcance (scope)
    varexterna = 444
    return foo
}


// Closures: no mutar variables externas, en su lugar hay que redefinirlas!
func externo() (func() int, int) {
    variablexterna := 2
    interno := func() int {
        variablexterna += 99 // intento de mutar variablexterna desde un scope externo
        return variablexterna // => 101 (pero la variablexterna está recién definida
                         //         variable sólamente visible desde interno)
    }
    return interno, variablexterna // => 101, 2 (variablexterna sigue siendo 2, no feu cambiada por foo!)
}
```

### Funciones variádicas
```go
func main() {
	fmt.Println(sumar(1, 2, 3)) 	// 6
	fmt.Println(sumar(9, 9))	// 18
	
	numeros := []int{10, 20, 30}
	fmt.Println(sumar(numeros...))	// 60
}

// Al usar ... antes del nombre del tipo del último parámetro, puedes indicar que puede tomar cero o más de esos parámetros.
// La función se invoca como cualquier otra, a excepción de que podemos pasar tantos argumentos como queramos. 
func sumar(args ...int) int {
	total := 0
	for _, v := range args { // Itera sobre args, sea cual sea su número de elementos.
		total += v
	}
	return total
}
```

## Tipos incorporados
```
bool

string

int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr

byte // alias para uint8

rune // alias para int32 ~= a character (Unicode code point) - very Viking

float32 float64

complex64 complex128
```

## Conversión de tipos
```go
var i int = 42
var f float64 = float64(i)
var u uint = uint(f)

// sintaxis alternativa
i := 42
f := float64(i)
u := uint(f)
```

## Paquetes 
* Declaración del paquete encima de cualquier archivo fuente
* Los ejecutables se encuentran en el *package `main`*
* Costumbre/convención: nombre del *package* == último nombre de la ruta del *import* (import `math/rand` => package `rand`)
* Identificador con mayúscula: exported (al ser exportado es visible desde otros packages)
* Identificador con minúscula: private (al ser privado, no es visible desde otros paquetes) 

## Estructuras de control

### If
```go
func main() {
	// Basic one
	if x > 0 {
		return x
	} else {
		return -x
	}
    	
	// Se puede poner una declaración antes de la condición 
	if a := b + c; a < 42 {
		return a
	} else {
		return a - 42
	}
    
	// Aserción de tipo dentro de if
	var val interface{}
	val = "foo"
	if str, ok := val.(string); ok {
		fmt.Println(str)
	}
}
```

### Bucles
```go
    // Sól existe: `for`, no existe `while`, ni `until`
    for i := 1; i < 10; i++ {
    }
    for ; i < 10;  { // bucle tipo while
    }
    for i < 10  { // se pueden omitir los punto y coma en caso de que haya sólo una condición 
    }
    for { // se puede omitir la condicón ~ mientras (verdadero)
    }
```

### Switch
```go
    // declaración de switch 
    switch sistemaOperativo {
    case "darwin":
        fmt.Println("Hipster de mac")
        // se rompe en automático, no sigue con las siguientes
    case "linux":
        fmt.Println("Geek de linux")
    default:
        // Windows, BSD, ...
        fmt.Println("Otro")
    }

    // al igual que con for e if, se puede tener una declaración antes del valor del switch  
    switch os := runtime.GOOS; os {
    case "darwin": ...
    }

    // también se pueden hacer comparaciones con los switch cases
    numero := 42
    switch {
        case numero < 42:
            fmt.Println("Menor")
        case numero == 42:
            fmt.Println("Igual")
        case numero > 42:
            fmt.Println("Mayor")
    }
```

## Arrays, Slices, Ranges

### Arrays
```go
var a [10]int // declara un array de int con una longitud de 10. La longitud del array es parte del tipo!

a[3] = 42     // asignar elementos
i := a[3]     // leer elementos

// declarar e inicializar
var a = [2]int{1, 2}
a := [2]int{1, 2} // anotación corta
a := [...]int{1, 2} // elipsis -> El compilador infiere la longitud del array
```

### Slices
```go
var a []int                              // declara un slice - similar a un array pero sin especificar su longitud 
var a = []int {1, 2, 3, 4}               // declara e inicializa un slice (soportado por un array implícito) 
a := []int{1, 2, 3, 4}                   // anotación corta
chars := []string{0:"a", 2:"c", 1: "b"}  // ["a", "b", "c"]

var b = a[base:tope]	// crea un slice (vista del array) del índice base a tope -1 
var b = a[1:4]		// slice de índice 1 al 3
var b = a[:3]		// a falta del índice base, este implica 0 
var b = a[3:]		// a falta del índice tope, este implica len(a)
a =  append(a,17,3)	// adjunta items al slice a
c := append(a,b...)	// concatena slices a y b 

// crear un slice con make
a = make([]byte, 5, 5)	// primer argumento es longitud, segundo capacidad
a = make([]byte, 5)	// la capacidad es opcional

// crear un slice a partir de un array
x := [3]string{"fresa", "mango", "naranja"}
s := x[:] // un slice haciendo referencia al almacenamiento de x 
```

### Operaciones con Arrays y Slices
`len(a)` te da la longitud de un array / un slice. Es una función incorporada, no un atributo/método del array. 

```go
// bucles/loops sobre arrays/slices
for i, e := range a {
    // i es el índice, e es el elemento
}

// Si sólo se necesita e:
for _, e := range a {
    // e es el elemento
}

// ...y si sólo se necesita el índice
for i := range a {
}

// En versiones de go antes de la 1.4, te arrojaba error al no usar tanto e como i.
// La versión 1.4 introdujo una forma libre de variables, por lo que se puede hacer esto:
for range time.Tick(time.Second) {
    // hazlo cada segundo
}

```

## Maps

```go
var m map[string]int
m = make(map[string]int) // Declaración con make

m["key"] = 42 //Asignación a "key" el valor 42

fmt.Println(m["key"]) // Imprime el valor asociado a "key"

delete(m, "key") // Elimina "key" del map "m"

elem, ok := m["key"] // verifica si su key de nombre "key" está presente y de ser así, la recupera

// map literal
var m = map[string]Vertex{
    "Bell Labs": {40.68433, -74.39967},
    "Google":    {37.42202, -122.08408},
}

```

## Structs

No existen las clases, sólo *structs*. Los *structs* pueden tener métodos. 
```go
// Un struct es un tipo/type. También es una colección de campos.

// Declaración
type Vertex struct {
    X, Y int
}

// Creación
var v = Vertex{1, 2}
var v = Vertex{X: 1, Y: 2} // Crea un struct al definir valores con sus keys/nombre asignado al campo.  
var v = []Vertex{{1,2},{5,2},{5,5}} // Inicializa un slice de structs

// Accediendo
v.X = 4

// Se pueden declarar métodos en structs. A la estructura que le quieras declarar
// el método (el type receptor) va entre la keyword func y 
// el nombre del método. La estrucutra se copia en cada llamada al método(!)
func (v Vertex) Abs() float64 {
    return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

// Llamado al método
v.Abs()

// Para mutar/hacer cambios a métodos, es necesario el uso de un pointer (ver abajo) hacia el struct
// como parte del tipo. Con esto, el valor del struct no se copia de la llamada el método. 
func (v *Vertex) add(n float64) {
    v.X += n
    v.Y += n
}

```
**structs anónimos:**  
Más seguro y mejor manejo que usar `map[string]interface{}`.
```go
point := struct {
	X, Y int
}{1, 2}
```

## Pointers
```go
p := Vertex{1, 2}  // p es un Vertex
q := &p            // q es un pointer a un Vertex
r := &Vertex{1, 2} // r es también un pointer a un Vertex

// El tipo de un pointer a un Vertex es *Vertex

var s *Vertex = new(Vertex) // new crea un pointer a una nueva instancia del struct
```

## Interfaces
```go
// declaración de interface
type Genializador interface {
    Genializar() string
}

// types *no* se declaran para implementar interfaces
type Foo struct {}

// en su lugar, los types satisfacen la interface de manera implítica si implementen todos sus métodos requeridos 
func (foo Foo) Genializar() string {
    return "Genial!"
}
```

## Incrustación
(embebidos)

No existen las subclases en Go. En su lugar, existen las interfaces y la incrustación de structs.

```go
// Las implementaciones de ReadWriter deben satisfacer tanto Reader como Writer
type ReadWriter interface {
    Reader
    Writer
}

// Server expone todos los métodos que tiene Logger
type Server struct {
    Host string
    Port int
    *log.Logger
}

// inicializa el tipo embebido/incrustado de la manera usual 
server := &Server{"localhost", 80, log.New(...)}

// los métodos implementados en el struct embebido pasan a través de 
server.Log(...) // llamados a server.Logger.Log(...)

// el nombre del campo del tipo embebido/incrustado es el nombre de su tipo/type (en este caso Logger) 
var logger *log.Logger = server.Logger
```

## Errores
No hay manejo de excepciones. Las funciones que pueden producir un error sólo declaran un valor adicional de retorno de tipo/typer `Error`.
Esta es la interface `Error`:

```go
type error interface {
    Error() string
}
```

Una función que puede retornar un error:
```go
func hacerAlgo() (int, error) {
}

func main() {
    resultado, error := doStuff()
    if (error != nil) {
        // manejo del error
    } else {
        // todo bien, usa el resultado
    }
}
```

# Concurrencia

## Goroutines
Las Goroutines son hilos ligeros (administrados por Go, no hilos del sistema operativo). `go f(a, b)` inicia una nueva goroutine que ejecuta `f` (dado que `f` es una función).

```go
// sólo una función (que posteriormente puede iniciarse como una goroutine)
func hacerAlgo(s string) {
}

func main() {
    // usando una función nombreada en una goroutine
    go hacerAlgo("foobar")

    // usando una función interna anónima en una goroutine 
    go func (x int) {
        // aquí va el cuerpo de la función
    }(42)
}
```

## Channels
```go
ch := make(chan int) // crea un channel de type/tipo int
ch <- 42             // Envía un valor al channel ch
v := <-ch            // Recibe un valor del channel ch

// Los channels sin buffer se bloquean. Se bloquea su lectura cuando no hay valor disponible y se bloquea su escritura si un valor ya ha sido escrito pero no leído.

// Crea un channel con buffer. La escritura a un channel con buffer no se bloquea si un menor número de elementos del tamaño del buffer han sido escritos, es decir, si aún tiene espacio.
ch := make(chan int, 100)

close(ch) // cierra el channel (sólo envío debería cerrarse)

// lee de un canal y verifica si ha sido cerrado
v, ok := <-ch

// si ok resulta false, el channel ha sido cerrado

// Leer de un canal hasta que esté cerrado 
for i := range ch {
    fmt.Println(i)
}

// Selecciona bloques en operaciones con canales múltiples, si uno se desbloquea, el case correspondiente se ejecuta.
func hacerAlgo(channelOut, channelIn chan int) {
    select {
    case channelOut <- 42:
        fmt.Println("Se podría escribir a channelOut!")
    case x := <- channelIn:
        fmt.Println("Se podría leer de channelIn")
    case <-time.After(time.Second * 1):
        fmt.Println("tiempo!")
    }
}
```

### Channel Axioms
- El envío a un channel nil se bloquea para siempre

  ```go
  var c chan string
  c <- "Hello, World!"
  // fatal error: all goroutines are asleep - deadlock!
  ```
- La recepción de un channel nil se bloquea para siempre

  ```go
  var c chan string
  fmt.Println(<-c)
  // fatal error: all goroutines are asleep - deadlock!
  ```
- El envío a un canal cerrado arroja panic

  ```go
  var c = make(chan string, 1)
  c <- "Hola mundo!"
  close(c)
  c <- "Hola panic!"
  // panic: send on closed channel
  ```
- La recepción de un channel cerrado retorna su valor cero de manera inmediata 

  ```go
  var c = make(chan int, 2)
  c <- 1
  c <- 2
  close(c)
  for i := 0; i < 3; i++ {
      fmt.Printf("%d ", <-c) 
  }
  // 1 2 0
  ```

## Impresión

```go
fmt.Println("Hola, 你好, नमस्ते, Привет, ᎣᏏᏲ") // impresión básica, además de agregar una nueva línea
p := struct { X, Y int }{ 17, 2 }
fmt.Println( "My point:", p, "x coord=", p.X ) // imprime structs, ints, etc.
s := fmt.Sprintln( "My point:", p, "x coord=", p.X ) // imprime variable de string

fmt.Printf("%d hex:%x bin:%b fp:%f sci:%e",17,17,17,17.0,17.0) // formato parecido a c
s2 := fmt.Sprintf( "%d %f", 17, 17.0 ) // impresión con formato de variable de string

hellomsg := `
 "Hola" en Chino es 你好 ('Ni Hao')
 "Hola" en Hindi es नमस्ते ('Namaste')
` // string multilínea, usando comilla al inicio y al final 
```

# Snippets

## HTTP Server
```go
package main

import (
    "fmt"
    "net/http"
)

// define un type para la respuesta
type Hola struct{}

// permite que el type implemente el método ServeHTTP (definido en la interface http.Handler)

func (h Hola) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    fmt.Fprint(w, "Hola!")
}

func main() {
    var h Hola
    http.ListenAndServe("localhost:4000", h)
}

// Aquí está el method signature de http.ServeHTTP:
// type Handler interface {
//     ServeHTTP(w http.ResponseWriter, r *http.Request)
// }
```


Traducido de [Golang cheat sheet de Ariel Mashraki](https://github.com/a8m/go-lang-cheat-sheet) \
Este texto es sólo una traducción y su contenido no me pertenece, a excepción de algunos comentarios y descripciones.
