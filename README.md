# Compresor de texto · Ensamblador MC88110

Compresor y descompresor de cadenas de texto escrito en **ensamblador del MC88110**. Usa un algoritmo de la familia **LZ77**: cada vez que un fragmento del texto ya ha aparecido antes, se sustituye por una referencia `(posición, longitud)` a esa aparición anterior.

> 🎓 **Proyecto académico**: práctica de la asignatura *Estructura de Computadores* (Grado en Ingeniería Informática)<br>
> Universidad Politécnica de Madrid · Campus de Montegancedo<br>
> 👥 Trabajo individual

---

## Formato del texto comprimido

```
┌──────────── cabecera (5 bytes) ────────────┬─ mapa de bits ─┬────────── zona de datos ──────────┐
│ LO (2 B) │ M (1 B) │ desplazamiento (2 B)  │ 1 bit / token  │ M·8 literales │ literal │ ref │ … │
└────────────────────────────────────────────┴────────────────┴───────────────────────────────────┘
```

| Campo | Descripción |
|-------|-------------|
| `LO` | Longitud del texto original (little endian) |
| `M` | Número de bloques de 8 caracteres que se copian tal cual al principio |
| Desplazamiento | Dónde empieza la zona de datos (`5 + bytes del mapa`) |
| Mapa de bits | Un bit por *token*: `0` = literal, `1` = referencia (bit más significativo primero) |
| Literal | 1 byte: el carácter tal cual |
| Referencia | 3 bytes: posición `P` (2 B) y longitud `L` (1 B) de la coincidencia |

Una coincidencia solo se codifica como referencia si mide **al menos 4 caracteres**, porque una referencia ocupa 3 bytes. La longitud máxima es de **255**.

## Subrutinas

Todas reciben los parámetros por la **pila** (`r30`), devuelven el resultado en `r29` y usan `r31` como puntero de marco.

| Subrutina | Qué hace |
|-----------|----------|
| `LongCad` | Longitud de una cadena terminada en `\0` |
| `BuscaCar` | Busca un carácter en una cadena entre dos posiciones |
| `CoincidenCad` | Número de caracteres iniciales iguales entre dos cadenas |
| `PoneBitA1` / `LeeBit` | Escribe o lee un bit concreto del mapa de bits |
| `BuscaMax` | Busca la coincidencia más larga anterior a una posición (*proporcionada por la asignatura*) |
| `Comprime` | Comprime un texto: construye el mapa de bits y la zona de datos en la pila, y después monta la salida |
| `Descomprime` | Reconstruye el texto original a partir del comprimido |
| `Verifica` | Comprime y descomprime un texto y comprueba que el resultado coincide con el original |

El programa principal (`ppal`) descomprime uno de los casos de prueba del enunciado.

## Ejecución

Se ensambla y ejecuta con el **emulador del MC88110** que proporciona la asignatura (ficheros `.ens`). El código empieza en `0x1000` y la pila se inicializa en `0xF000`.

## Correcciones respecto a la entrega

Al revisar el código para publicarlo se corrigieron dos fallos que no se manifestaban con los casos de prueba de la entrega:

- **Mapa de bits sin inicializar.** Al empezar cada byte nuevo del mapa había que ponerlo a `0`, pero la condición (`bb0 ne, r10`) miraba un bit del propio valor y no el resultado de una comparación, así que nunca se limpiaba. Funcionaba solo porque la memoria del simulador empieza a cero.
- **Textos de menos de 8 caracteres.** `Descomprime` copiaba siempre `M·8` caracteres iniciales aunque el texto fuera más corto, y el `\0` final quedaba en una posición incorrecta.
