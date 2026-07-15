# Condiciones

En Infernal, las condiciones funcionan de una forma similar a las de **Lua**. Parece que hablas en inglés con el intérprete de Infernal :).

## ¿Para qué sirven?
Las condiciones permiten controlar el flujo de ejecución de un programa dependiendo del resultado de una comparación.
Por ejemplo, una variable puede contener información que determine qué acción debe realizar el programa. Mediante una condición, es posible comprobar el valor de esa variable y ejecutar un bloque de código específico cuando se cumpla una determinada condición.


## ¿Cómo se usan?
En Infernal, las condiciones tienen la siguiente estructura:
```Infernal
if ALGO operador algo then
    código
fi
```
Una condición comienza con `if`, utiliza `then` para indicar el inicio del bloque de código que se ejecutará si la condición es verdadera, y finaliza con `fi`.
Dentro de una condición existen tres elementos principales:
`ALGO`: es el primer valor de la comparación. Puede ser una variable o un valor directo, como un `int` o un `string`.
`operador`: define la comparación que se realizará entre los valores. Los operadores se explican en detalle en la documentación de Operadores.
`algo`: es el segundo valor utilizado en la comparación.


## Ejemplo de uso:
``` Infernal
int x = 6

if x != 6 then
    print("Tu número no es 6")
fi
```
El operador `!=` significa "es diferente a". Por lo tanto, este `if` se puede leer como:
"Si x es diferente a 6, entonces..."
En este caso, la condición no se cumple porque el valor de `x` es 6, por lo que el mensaje no se mostrará.

### Ejemplo de uso real
Las condiciones pueden utilizarse para comprobar información antes de ejecutar una acción. Por ejemplo, un programa puede verificar si se está ejecutando con los permisos necesarios:
``` Infernal
user = whoami

if user != "root" then
    error("Tienes que ejecutarlo como root (sudo)")
    exit(2)
fi
```
En este ejemplo, el programa obtiene el usuario actual y comprueba si no es root. Si la condición se cumple, muestra un error y finaliza la ejecución con el código de salida 2 (argumentos/uso incorrecto del programa).