# carta-marina-cordoba

Proyecto para la extracción, procesamiento y análisis de datos de las Cartas Marinas de Córdoba (2013–2023).

## Origen del proyecto

Este proyecto toma como punto de partida los trabajos desarrollados por **Open Data Córdoba**, disponibles en los siguientes repositorios:

- **Elecciones Provinciales 2015**  
  https://github.com/OpenDataCordoba/elecciones2015/

- **Carta Marina 2017**  
  https://github.com/avdata99/carta-marina-2017/

A partir de dichos proyectos se desarrollaron y adaptaron los parsers necesarios para procesar las distintas Cartas Marinas, con los siguientes objetivos:

- Adaptarlos a Python 3.
- Ejecutarlos en Google Colab.
- Documentar las modificaciones realizadas respecto de los parsers originales y entre las distintas ediciones de la Carta Marina.
- Unificar el modelo de datos generado para todos los años.
- Preservar la mayor cantidad posible de información durante la extracción.
- Facilitar la incorporación de nuevas Cartas Marinas en futuras elecciones.

Las adaptaciones se documentan a continuación en orden cronológico según el año de cada Carta Marina, independientemente del orden en que fueron desarrollados los parsers.

---

# Parser 2013

## Objetivo de esta adaptación

El parser correspondiente a 2013 fue desarrollado a partir de la lógica utilizada para procesar las Cartas Marinas posteriores, incorporando las modificaciones necesarias para adaptarla a la estructura particular de este documento.

La Carta Marina 2013 presenta diferencias importantes en la identificación de las secciones y circuitos electorales y en la disposición de las columnas correspondientes a los establecimientos.

## Diferencias respecto de los formatos posteriores

### 1. Detección de secciones

#### Formato utilizado en años posteriores

En las Cartas Marinas posteriores, las secciones se identifican explícitamente mediante líneas del tipo:

```text
Sección 1 - CAPITAL
```

Esto permite obtener directamente tanto el número como el nombre de la sección.

#### Formato 2013

En 2013, al comienzo de cada sección aparece únicamente su nombre:

```text
CAPITAL
```

Al finalizar la sección aparece:

```text
RESUMEN DE LA SECCION: 1 - CAPITAL
```

y posteriormente comienza la siguiente:

```text
CALAMUCHITA
```

#### Adaptación

Se incorporó una variable de estado que permite identificar cuándo el parser se encuentra esperando el nombre de una nueva sección.

Al comenzar el documento y después de encontrar:

```text
RESUMEN DE LA SECCION
```

la siguiente línea válida se interpreta como el nombre de la nueva sección. El número de sección se obtiene incrementalmente siguiendo el orden del propio documento.

#### Justificación

Esta solución permite conservar la estructura general del parser sin incorporar una lista fija de departamentos.

De esta manera, la identificación de las secciones continúa dependiendo de la estructura de la Carta Marina y no de valores previamente definidos en el código.

### 2. Detección de circuitos

#### Formato utilizado en años posteriores

En las Cartas Marinas posteriores los circuitos se identifican mediante líneas del tipo:

```text
Circuito 1 - SECCIONAL PRIMERA
```

#### Formato 2013

En 2013 se elimina la palabra `Circuito` y el encabezado adopta estructuras como:

```text
1 - SECCIONAL PRIMERA
4A - PUEBLO LAS FLORES
14Q - VILLA WARCALDE
```

Además, en la misma línea aparecen los encabezados correspondientes a las columnas de los establecimientos.

#### Adaptación

Se modificó la detección de circuitos para reconocer códigos numéricos o alfanuméricos seguidos por un guion.

Una vez identificado el circuito, se separa su nombre de los encabezados de columnas que aparecen posteriormente en la misma línea.

#### Justificación

La adaptación permite reconocer los circuitos a partir de su estructura sin depender de la palabra `Circuito`, inexistente en el formato de 2013.

También evita incorporar los encabezados `Desde/Hasta`, `Mesas` y `Electores` dentro del nombre del circuito.

### 3. Cambio en la estructura de las columnas

Las filas correspondientes a establecimientos presentan conceptualmente la siguiente estructura:

```text
Establecimiento    Desde/Hasta    Mesas    Electores
```

Por ejemplo:

```text
CENTRO EDUC.NIVEL MEDIO ADULTO DEAN FUNES 417    0001 / 0007    7    2.408
```

#### Adaptación

La extracción se realiza interpretando las columnas desde el final de cada fila:

- Última columna → cantidad de electores.
- Penúltima columna → cantidad de mesas.
- Antepenúltima columna → rango de mesas.
- Todo lo anterior → establecimiento.

El rango de mesas utiliza `/` como separador:

```text
0001 / 0007
```

#### Justificación

Esta modificación permite conservar el mismo modelo de datos utilizado para los demás años aunque la disposición de la información en el documento original sea diferente.

### 4. Inconsistencia de espaciado en una fila

#### Problema detectado

Durante el procesamiento se encontró una fila en la que la cantidad de mesas y la cantidad de electores están separadas por tres espacios en lugar de los cuatro utilizados como delimitador general:

```text
INSTITUTO DEL ROSARIO BV.ALVEAR 68    4086 / 4115    30   10.320
```

Como consecuencia, ambos valores son interpretados inicialmente como un único elemento:

```text
30   10.320
```

#### Adaptación

Se incorporó un tratamiento específico para las filas que generan únicamente tres componentes. En estos casos se separa el último componente para recuperar individualmente la cantidad de mesas y la cantidad de electores.

#### Justificación

El diagnóstico del archivo permitió determinar que se trata de una anomalía puntual de formato.

Por este motivo se mantuvo el criterio general de separación utilizado por el parser y se agregó únicamente el tratamiento necesario para esta excepción.

### 5. Validación de continuidad de mesas

Se mantiene el control de continuidad utilizado en los demás parsers. Las discontinuidades detectadas se registran para su posterior revisión, pero no interrumpen la ejecución.

En 2013 se observan numerosos casos en los que los establecimientos no están presentados siguiendo estrictamente el orden numérico de las mesas. Por este motivo, una discontinuidad registrada no implica necesariamente la ausencia de una mesa, sino que puede deberse al orden de presentación del documento.

## Validación

El resultado obtenido coincide con el resumen oficial de la Carta Marina 2013:

| Concepto | Carta Marina | Parser |
|----------|-------------:|-------:|
| Establecimientos | 1.163 | 1.163 |
| Mesas | 7.986 | 7.986 |
| Electores | 2.645.525 | 2.645.525 |

La coincidencia de los totales permite validar que las particularidades de formato detectadas no provocaron pérdida de registros durante la extracción.

## Conclusión

La Carta Marina 2013 presenta una estructura diferente de la utilizada en los documentos posteriores, principalmente en la identificación de secciones y circuitos y en la disposición de las columnas.

Las adaptaciones realizadas permiten conservar el mismo modelo de datos de salida utilizado para el resto del proyecto sin depender de listas predefinidas de secciones o circuitos.

---

# Parser 2015

## Objetivo de esta adaptación

El objetivo fue mantener la estructura y el funcionamiento del algoritmo original, realizando únicamente las modificaciones necesarias para:

- Adaptarlo a Python 3.
- Ejecutarlo en Google Colab.
- Resolver incompatibilidades detectadas durante el procesamiento de la Carta Marina 2015.
- Preservar la mayor cantidad posible de información durante la extracción.
- Documentar las decisiones de diseño adoptadas durante el proceso.

## Diferencias respecto del repositorio original

El parser fue desarrollado tomando como base el repositorio original correspondiente a las elecciones provinciales de 2015.

El objetivo no fue reescribir el algoritmo, sino reproducir su funcionamiento realizando únicamente las modificaciones necesarias para adaptarlo a Python 3, corregir incompatibilidades detectadas durante la ejecución en Google Colab y resolver particularidades presentes en la Carta Marina 2015.

### 1. Separación de líneas

#### Original

El parser original divide el archivo utilizando:

```python
lines = raw.split("\n")
```

#### Adaptación

Se reemplazó por:

```python
lines = raw.splitlines()
```

#### Justificación

`splitlines()` interpreta correctamente los distintos caracteres de fin de línea generados por `pdftotext`, incluyendo los saltos de página (`\f`), evitando líneas residuales sin modificar el contenido del documento.

### 2. Validación de continuidad de mesas

#### Original

Ante una discontinuidad en la numeración de mesas, el parser finaliza inmediatamente mediante:

```python
exit(1)
```

#### Adaptación

Las discontinuidades ya no interrumpen el procesamiento. En su lugar se registran para su posterior revisión.

#### Justificación

Durante el procesamiento de la Carta Marina 2015 se detectaron excepciones legítimas en la numeración de mesas, como mesas asociadas fuera de secuencia.

Registrar estas situaciones permite completar la extracción sin perder la información necesaria para su validación posterior.

### 3. Manejo de saltos de página entre circuitos

#### Problema detectado

En la Carta Marina 2015 existen circuitos cuyo encabezado aparece al final de una página y cuyo listado de establecimientos continúa en la página siguiente.

El parser original considera que una línea vacía finaliza el bloque de establecimientos. En estos casos, el primer establecimiento del circuito queda omitido.

Ejemplo:

```text
Circuito 4D

(Página siguiente)

COL VILLA EUCARISTICA...
```

#### Adaptación

Las líneas vacías pasan a ignorarse sin abandonar el estado de lectura de establecimientos.

El estado únicamente finaliza al encontrar:

```text
Resúmen del Circuito
```

#### Justificación

Esta modificación permite atravesar correctamente los saltos de página sin perder registros.

### 4. Reconstrucción del establecimiento

#### Problema detectado

Algunas direcciones contienen secuencias de cuatro espacios utilizadas también como delimitador de columnas.

Ejemplo:

```text
ESC PANAMERICANA - SOTO 919    - B°ACOSTA    9    00337 a 00345    3.078
```

El parser original interpreta esta línea como cinco columnas en lugar de cuatro.

#### Adaptación

En lugar de asumir que el establecimiento ocupa una única columna, se reconstruye utilizando todos los elementos previos a las tres últimas columnas.

Conceptualmente:

- Última columna → cantidad de electores.
- Penúltima → rango de mesas.
- Antepenúltima → cantidad de mesas.
- Todo lo anterior → establecimiento.

#### Justificación

El README del repositorio original ya advierte que las direcciones presentan inconsistencias y pueden requerir correcciones manuales.

Esta adaptación automatiza uno de esos casos sin modificar la estructura general del parser.

## Resultado

El parser mantiene la estructura general del algoritmo original, introduciendo únicamente las modificaciones necesarias para:

- Ejecutarse correctamente en Python 3.
- Funcionar en un entorno moderno como Google Colab.
- Procesar correctamente las particularidades detectadas en la Carta Marina 2015 sin intervención manual durante la extracción.

Las tareas de limpieza y normalización de los datos se realizan en una etapa posterior, preservando el parser como un proceso dedicado exclusivamente a la extracción de información.

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

Esta modificación permite reutilizar el mismo parser para las Cartas Marinas posteriores sin depender del año específico indicado en el encabezado.

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

Esta condición evita depender del año específico indicado en el encabezado y facilita la reutilización del parser.

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

#### Adaptación

Se modificó la regla utilizada para reconocer el encabezado y extraer el número y nombre del circuito.

#### Justificación

El cambio permite mantener la misma información de salida pese a la modificación del formato del documento.

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

#### Adaptación

Se modificó la condición utilizada para detectar el final del bloque correspondiente a cada circuito.

### 5. Conservación del campo establecimiento

Al igual que en los parsers anteriores, el establecimiento continúa almacenándose como un único campo de texto, preservando íntegramente la información presente en la Carta Marina.

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
