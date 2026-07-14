# Documentación de Infernal hecha por IA

Infernal es un lenguaje de *scripting* interpretado, implementado en C, orientado a automatizaciones de línea de comandos. Su sintaxis combina bloques cerrados con `fi`, expresiones y funciones sencillas, y la ejecución directa de comandos del sistema.

Esta documentación describe el comportamiento de la versión **1.0.1** incluida en este repositorio.

## Inicio rápido

### Requisitos y compilación

Se necesita `make`, un compilador C compatible con C11 y las utilidades habituales de un sistema POSIX. Para generar el intérprete:

```sh
make
```

El binario resultante es `./infernal`.

```sh
./infernal --help
./infernal --version
./infernal ruta/al/script.infernal argumentos...
```

Sin argumentos se muestra el mensaje de bienvenida. `--edition` muestra también la edición embebida.

### Primer programa

Guarda este contenido como `hola.infernal`:

```infernal
string nombre = "Mundo"
print("Hola, " + nombre + "!")
```

Ejecútalo con:

```sh
./infernal hola.infernal
```

## Sintaxis básica

Las sentencias se separan por líneas. Los comentarios de una línea empiezan con `#`; para un comentario de bloque se usa `###` como apertura y cierre.

```infernal
# Comentario de una línea
###
  Comentario de varias líneas.
###
```

Los bloques de `if`, bucles, funciones y manejo de errores terminan con `fi`.

### Valores y variables

Los tipos disponibles son `int`, `float`, `bool`, `string` y `list`. Una declaración tipada fija el tipo de la variable; una asignación sin tipo infiere el tipo del valor inicial.

```infernal
int cantidad = 3
float precio = 4.5
bool activo = true
string saludo = "hola"
list tareas = ["compilar", "probar"]

total = cantidad * precio
```

`local` declara en el ámbito actual y `global` declara una variable global que además se exporta al entorno del proceso como `INFERNAL_VAR_<nombre>`.

```infernal
local string temporal = "solo en este ámbito"
global string entorno = "visible para procesos hijos"
```

Las variables se consultan por su nombre en expresiones. El prefijo `$` se acepta sobre todo para interpolarlas en comandos del sistema:

```infernal
print(saludo)
echo $saludo
```

### Expresiones y operadores

Infernal soporta los operadores aritméticos `+`, `-`, `*`, `/` y `%`; comparaciones `==`, `!=`, `<=` y `>=`; y lógica `and`/`&&`, `or`/`||`.

```infernal
int a = 7
int b = 2
print(a + b)          # 9
print(a / b)          # 3.5
print(a >= b and true)
```

El operador `+` concatena si alguno de los operandos es una cadena. Las listas usan índices empezando en **1**:

```infernal
list colores = ["rojo", "verde", "azul"]
print(colores[1])
colores[2] = "amarillo"
```

## Control de flujo

### Condicionales

```infernal
if cantidad >= 10 then
    print("stock suficiente")
elseif cantidad >= 1 then
    warn("queda poco stock")
else
    error("sin stock")
fi
```

### Bucles

`while` repite mientras la condición sea verdadera. `break` abandona el bucle y `continue` pasa a la siguiente iteración.

```infernal
int i = 1
while i <= 3 then
    print(i)
    i = i + 1
fi
```

Hay dos formas de `for`:

```infernal
# Inicialización; condición; incremento
for int i = 0; i <= 2; i = i + 1 then
    print(i)
fi

# Recorrido de una lista
for tarea in ["build", "test"] then
    print(tarea)
fi
```

El intérprete limita las iteraciones para evitar bucles accidentales. Se puede consultar o cambiar el límite con `getlooplimit()` y `setlooplimit(entero)`.

### Funciones

Las funciones se definen con `function` y pueden tener parámetros tipados. Una función debe estar definida o importada antes de invocarse.

```infernal
function area(float base, float altura)
    return base * altura
fi

print(area(3, 2.5))
```

`return` sin expresión devuelve un valor nulo. Los parámetros omitidos reciben nulo.

### Errores recuperables

`try` ejecuta un bloque y, si éste genera un error de Infernal o un comando falla, continúa con `catch`.

```infernal
try
    comando_que_puede_fallar
catch
    warn("no se pudo completar la operación")
fi
```

También puede usarse `or` entre comandos para expresar un bloque de recuperación breve:

```infernal
comando_principal or comando_alternativo
```

## Biblioteca estándar

| Función | Uso |
| --- | --- |
| `print(...)` | Escribe los valores en salida estándar. |
| `warn(...)` | Escribe en amarillo. |
| `error(...)` | Escribe en rojo; no genera una excepción. |
| `success(...)` | Escribe en verde. |
| `input([mensaje])` | Muestra el mensaje opcional y devuelve una línea leída de stdin. |
| `vartype(valor)` | Devuelve `int`, `float`, `bool`, `string`, `list` o `null`. |
| `printAllVars()` | Muestra las variables accesibles y sus tipos. |
| `lower(cadena)` | Devuelve la cadena en minúsculas. |
| `here()` | Devuelve el directorio del script en ejecución. |
| `getlooplimit()` | Devuelve el límite de iteraciones. |
| `setlooplimit(entero)` | Cambia el límite de iteraciones. |
| `exit([código])` | Finaliza el intérprete. |

## Comandos del sistema y captura de salida

Una línea que no se interpreta como sentencia de Infernal se ejecuta mediante el *shell* del sistema. Las variables con `$nombre` se expanden antes de ejecutar el comando.

```infernal
string directorio = here()
echo "El script está en $directorio"
```

Para capturar la salida de un comando, asígnalo a una variable. La última nueva línea se elimina:

```infernal
string fecha = date +%F
print("Hoy es " + fecha)
```

Ten en cuenta que los comandos pasan por `system()`/`popen()`: valida siempre los datos que procedan de usuarios antes de interpolarlos en un comando.

## Módulos `.fire` y binarios embebidos

Durante la compilación, el `Makefile` incorpora automáticamente:

- Cada `config/infernal/fire/*.fire` como módulo de Infernal.
- Cada archivo de `config/infernal/bins/*` como binario ejecutable embebido.

Un módulo `.fire` se importa por su nombre base y sus funciones se exponen con el prefijo del módulo:

```infernal
import test
print(test.sumar(2, 3))
```

También se pueden importar ficheros por ruta entre comillas:

```infernal
import "./mi_modulo.fire"
```

Los módulos identificados sin comillas que no estén embebidos se buscan en `/usr/share/infernal/fire/<nombre>.fire`.

Para ejecutar un binario embebido se rodea el comando con `!`:

```infernal
!hi!
```

Los argumentos pueden interpolar variables:

```infernal
string mensaje = "hola"
!mi_bin $mensaje!
```

Los binarios se escriben temporalmente con permisos de ejecución y se eliminan después de ejecutarse. Tras cambiar `config/`, vuelve a ejecutar `make` para regenerar el intérprete.

## Argumentos de scripts: `flags`

La sentencia `flags(modo, ...)` asocia argumentos de la línea de comandos con bloques. En modo `0` reconoce nombres de opción; se admiten alias separados por `|`, valores tipados y un bloque comodín `*`.

```infernal
flags(0,
    --verbose | -v {
        print("modo detallado")
    },
    --nombre | -n = string nombre {
        print("Hola, " + nombre)
    },
    * {
        warn("argumento no reconocido: " + _)
    }
)
```

Ejemplo:

```sh
./infernal cli.infernal --nombre Ada -v
```

El modo `1` interpreta las especificaciones por posición, en el orden definido. El bloque comodín recibe los argumentos restantes en la variable `_`.

## Portales y repetición desde una línea

Los portales son marcadores de ejecución para reiniciar el programa desde la sentencia posterior. Se declaran con `@nombre` y se salta con `repeat @nombre`.

```infernal
@inicio
print("vuelta")
# repeat @inicio reiniciaría desde print("vuelta")
```

`repeat line <entero>` realiza el mismo salto usando un número de línea fuente. Esta característica altera el flujo del bloque actual; úsala con una condición de salida para evitar ciclos infinitos. `local @nombre` limita el portal al ámbito actual.

## Arquitectura del repositorio

```text
src/
├── main.c          punto de entrada y configuración del entorno
├── lexer/          tokenización y palabras reservadas
├── parser/         construcción del AST y análisis de expresiones/flags
├── core/           tipos, valores, AST y utilidades de memoria
├── runtime/        ámbitos, evaluador, errores y ejecución de comandos
├── stdlib/          funciones integradas y módulos embebidos
└── metadata/        textos de versión, ayuda, bienvenida y edición
config/infernal/
├── fire/           módulos `.fire` que se embeberán al compilar
└── bins/           ejecutables o scripts binarios embebidos
```

El flujo de ejecución es: `main` inicializa ámbitos y funciones integradas → el *lexer* genera tokens → el *parser* construye un AST → el evaluador ejecuta los nodos. El `Makefile` genera fuentes C temporales para los recursos de `config/` y `src/metadata/`, los compila y los enlaza dentro de `infernal`.

## Desarrollo y limpieza

```sh
make          # compila el intérprete y los recursos embebidos
make clean    # elimina build/ e infernal
make distclean
make help
```

La licencia del proyecto es GPL-3.0-or-later; consulta [`LICENSE`](LICENSE).
