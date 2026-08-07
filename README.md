# carta-marina-cordoba

Proyecto para la extracción, procesamiento y análisis de datos de las Cartas Marinas de Córdoba (2015–2023).

## Origen del proyecto

Este proyecto toma como punto de partida los trabajos desarrollados por **Open Data Córdoba**, disponibles en los siguientes repositorios:

- **Elecciones Provinciales 2015**
  https://github.com/OpenDataCordoba/elecciones2015/

- **Carta Marina 2017**
  https://github.com/avdata99/carta-marina-2017/

A partir de dichos proyectos se desarrolló una nueva implementación de los parsers con los siguientes objetivos:

- Adaptarlos a Python 3.
- Ejecutarlos en Google Colab.
- Documentar todas las modificaciones realizadas respecto de los parsers originales.
- Unificar el modelo de datos generado para todas las Cartas Marinas.
- Preservar la mayor cantidad posible de información durante la extracción.
- Facilitar la incorporación de nuevas Cartas Marinas en futuras elecciones.

---
# Parser 2019


## Objetivo de esta adaptación

Luego de la adaptación realizada sobre el parser de 2017, se buscó adaptar el algoritmo al nuevo formato incorporado por la Carta Marina 2019, preservando en la mayor medida posible la lógica original y el modelo de datos desarrollado para las versiones anteriores.


## Diferencias respecto del parser 2017

### 1. Cambio de formato de la Carta Marina

A diferencia de las Cartas Marinas 2015 y 2017, el documento correspondiente a 2019 presenta modificaciones importantes en su estructura.

Entre los principales cambios se encuentran:

- Nuevo encabezado de la elección.
- Nuevo formato para la identificación de los circuitos.
- Cambios en el resumen de cada circuito.
- Reorganización visual de las columnas del listado de establecimientos.

Estas modificaciones obligan a adaptar distintas reglas del parser, aunque se mantiene la misma lógica general de extracción.



### 2. Detección del encabezado de la elección

#### Formato anterior

Las Cartas Marinas anteriores utilizaban encabezados como:

```text
ELECCIONES 2015
ELECCIONES 2017
```

#### Adaptación

La detección del encabezado se generalizó utilizando:

```python
if "ELECCIONES 20" in linea:
```

#### Justificación

Esta condición permite reutilizar el mismo parser para todas las Cartas Marinas comprendidas entre 2015 y 2023, independientemente del año específico indicado en el encabezado.


### 3. Detección de circuitos

#### Formato anterior

En 2015 y 2017 los circuitos se identificaban mediante líneas del tipo:

```text
 Circuito   1 - SECCIONAL PRIMERA
```

#### Formato 2019

En 2019 el encabezado adopta el formato:

```text
Circuito:1 - SECCIONAL PRIMERA
```

#### Justificación

Fue necesario adaptar la detección y extracción del número y nombre del circuito al nuevo formato manteniendo la misma información de salida.



### 4. Cambio en el resumen de circuito

#### Formato anterior

Los documentos anteriores utilizaban:

```text
Resúmen del Circuito
```

#### Formato 2019

El documento utiliza:

```text
Resumen del Circuito:
```

#### Justificación

Se adaptó la condición utilizada para detectar el final del bloque correspondiente a cada circuito.


### 5. Conservación del campo establecimiento

Al igual que en el parser desarrollado para 2015, el establecimiento continúa almacenándose como un único campo de texto, preservando íntegramente la información presente en la Carta Marina.

No se intenta separar durante la extracción:

- nombre del establecimiento;
- dirección;
- barrio;
- localidad.

Esta normalización se realiza posteriormente sobre el archivo CSV.

#### Justificación

Las diferencias de formato entre establecimientos hacen que una separación temprana incremente la complejidad del parser y pueda provocar pérdida de información.


## Validación

El resultado obtenido coincide exactamente con el resumen oficial de la Carta Marina 2019:

| Concepto | Carta Marina | Parser |
|----------|-------------:|-------:|
| Establecimientos | 1.211 | 1.211 |
| Mesas | 8.649 | 8.649 |
| Electores | 2.884.358 | 2.884.358 |

Durante el procesamiento se detectaron cinco discontinuidades en la numeración de mesas. Estas corresponden a registros que aparecen fuera de secuencia dentro del documento y no implican pérdida de información.

En consecuencia, el parser recupera la totalidad de la información contenida en la Carta Marina 2019.



## Conclusión

La Carta Marina 2019 representa el primer cambio significativo de formato dentro de la serie analizada.

A diferencia de la transición entre 2015 y 2017, que requirió modificaciones mínimas, el documento de 2019 obligó a adaptar diversas reglas de detección y extracción debido a los cambios en la estructura del PDF.

No obstante, fue posible conservar el mismo modelo de datos de salida y la lógica general del parser, obteniendo una extracción completa y consistente con el resumen oficial de la Carta Marina.

Las adaptaciones implementadas permitieron mantener un único flujo de procesamiento para las distintas Cartas Marinas, modificando únicamente aquellos componentes dependientes del formato del documento.

---
# Parser 2017

## Objetivo de esta adaptación

Luego de la adaptación realizada sobre el parser de 2015, se buscó comprobar si dichas mejoras permitían reutilizar el mismo algoritmo sobre la Carta Marina 2017 sin introducir modificaciones específicas.

## Diferencias respecto del parser 2015

### 1. Generalización del encabezado de la elección

#### Original

El parser identificaba el encabezado mediante una condición específica para el año 2015:

```python
if "ELECCIONES 2015" in linea:
```

#### Adaptación

Se reemplazó por:

```python
if "ELECCIONES 20" in linea:
```

#### Justificación

Esta modificación permite reutilizar el mismo parser para todas las Cartas Marinas comprendidas entre 2015 y 2023 sin depender del año específico indicado en el encabezado.


## Validación

El resultado obtenido coincide exactamente con el resumen oficial de la Carta Marina 2017:

| Concepto | Carta Marina | Parser |
|----------|-------------:|-------:|
| Establecimientos | 1.211 | 1.211 |
| Mesas | 8.649 | 8.649 |
| Electores | 2.884.358 | 2.884.358 |

Durante el procesamiento se detectaron cinco discontinuidades en la numeración de mesas. Al igual que en 2015, estas corresponden a registros que aparecen fuera de secuencia dentro del documento y no implican pérdida de información.

## Conclusión

Las adaptaciones incorporadas durante el desarrollo del parser para la Carta Marina 2015 resultaron suficientes para procesar correctamente la Carta Marina 2017.

La única modificación necesaria consistió en generalizar la detección del encabezado correspondiente al año de la elección, confirmando que el algoritmo puede reutilizarse entre distintas Cartas Marinas con cambios mínimos.

---
# Parser 2015


## Objetivo de esta adaptación

El objetivo fue mantener la estructura y el funcionamiento del algoritmo original, realizando únicamente las modificaciones necesarias para:

- Adaptarlo a Python 3.
- Ejecutarlo en Google Colab.
- Resolver incompatibilidades detectadas durante el procesamiento de la Carta Marina 2015.
- Preservar la mayor cantidad posible de información durante la extracción.
- Documentar todas las decisiones de diseño adoptadas durante el proceso.
## Diferencias respecto del repositorio original

Este parser fue desarrollado tomando como base el repositorio original de la Carta Marina 2015.

El objetivo no fue reescribir el algoritmo, sino reproducir su funcionamiento realizando únicamente las modificaciones necesarias para adaptarlo a Python 3, corregir incompatibilidades detectadas durante la ejecución en Google Colab y resolver particularidades presentes en la Carta Marina 2015.



## 1. Separación de líneas

### Original

El parser original divide el archivo utilizando:

```python
lines = raw.split("\n")
```

### Adaptación

Se reemplazó por:

```python
lines = raw.splitlines()
```

### Justificación

`splitlines()` interpreta correctamente los distintos caracteres de fin de línea generados por `pdftotext`, incluyendo los saltos de página (`\f`), evitando líneas residuales sin modificar el contenido del documento.



## 2. Validación de continuidad de mesas

### Original

Ante una discontinuidad en la numeración de mesas, el parser finaliza inmediatamente mediante:

```python
exit(1)
```

### Adaptación

Las discontinuidades ya no interrumpen el procesamiento.

En su lugar se registran para su posterior revisión.

### Justificación

Durante el procesamiento de la Carta Marina 2015 se detectaron excepciones legítimas en la numeración de mesas, como mesas asociadas fuera de secuencia.

Registrar estas situaciones permite completar la extracción sin perder la información necesaria para su validación posterior.


## 3. Manejo de saltos de página entre circuitos

### Problema detectado

En la Carta Marina 2015 existen circuitos cuyo encabezado aparece al final de una página y cuyo listado de establecimientos continúa en la página siguiente.

El parser original considera que una línea vacía finaliza el bloque de establecimientos. En estos casos, el primer establecimiento del circuito queda omitido.

Ejemplo:

```
Circuito 4D

(Página siguiente)

COL VILLA EUCARISTICA...
```

### Adaptación

Las líneas vacías pasan a ignorarse sin abandonar el estado de lectura de establecimientos.

El estado únicamente finaliza al encontrar:

```
Resúmen del Circuito
```

### Justificación

Esta modificación permite atravesar correctamente los saltos de página sin perder registros.



## 4. Reconstrucción del establecimiento

### Problema detectado

Algunas direcciones contienen secuencias de cuatro espacios utilizadas también como delimitador de columnas.

Ejemplo:

```
ESC PANAMERICANA - SOTO 919    - B°ACOSTA    9    00337 a 00345    3.078
```

El parser original interpreta esta línea como cinco columnas en lugar de cuatro.

### Adaptación

En lugar de asumir que el establecimiento ocupa una única columna, se reconstruye utilizando todos los elementos previos a las tres últimas columnas.

Conceptualmente:

- Última columna → Cantidad de electores.
- Penúltima → Rango de mesas.
- Antepenúltima → Cantidad de mesas.
- Todo lo anterior → Establecimiento.

### Justificación

El README del repositorio original ya advierte que las direcciones presentan inconsistencias y pueden requerir correcciones manuales.

Esta adaptación automatiza uno de esos casos sin modificar la estructura general del parser.



## Resultado

El parser mantiene la estructura general del algoritmo original, introduciendo únicamente las modificaciones necesarias para:

- Ejecutarse correctamente en Python 3.
- Funcionar en un entorno moderno como Google Colab.
- Procesar correctamente las particularidades detectadas en la Carta Marina 2015 sin intervención manual durante la extracción.

Las tareas de limpieza y normalización de los datos se realizan en una etapa posterior, preservando el parser como un proceso dedicado exclusivamente a la extracción de información.
