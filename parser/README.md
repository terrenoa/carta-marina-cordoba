# carta-marina-cordoba

Proyecto para la extracción y procesamiento de datos de las Cartas Marinas de Córdoba correspondientes a las elecciones provinciales de 2013, 2015, 2017, 2019 y 2023.

El objetivo es transformar los documentos originales en archivos CSV estructurados y comparables entre elecciones, preservando la información contenida en cada Carta Marina.

Los archivos generados pueden utilizarse posteriormente como entrada para otras etapas de procesamiento, como normalización de domicilios, geolocalización o análisis espacial.

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

Las tareas posteriores de normalización o geolocalización se realizan de manera independiente sobre los archivos CSV generados.

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

---

# Parser 2023

## Objetivo de esta adaptación

La Carta Marina 2023 presenta un nuevo cambio de formato respecto de la edición de 2019.

Si bien se mantiene la información general de secciones, circuitos, establecimientos, mesas y electores, la disposición del documento cambia lo suficiente como para requerir una nueva lógica de extracción.

El objetivo de esta adaptación fue reconstruir esa información manteniendo el mismo modelo de datos utilizado en los años anteriores y preservando el contenido original de los establecimientos.

## Diferencias respecto del parser 2019

### 1. Cambio en la estructura de los establecimientos

En la Carta Marina 2023 los establecimientos ya no pueden procesarse suponiendo que toda su información se encuentra contenida en una única línea.

Un registro puede ocupar varias líneas dependiendo de la extensión del nombre y domicilio del establecimiento.

El rango de mesas, el tipo de mesa y la cantidad de electores permiten identificar el comienzo de cada registro.

#### Adaptación

El parser identifica el comienzo de un establecimiento mediante una expresión regular que reconoce:

- mesa inicial;
- mesa final;
- tipo de mesa;
- cantidad de electores.

El texto anterior al rango se considera el comienzo del campo `escuela`.

Las líneas posteriores que no representan un nuevo establecimiento, circuito, sección o encabezado se agregan al establecimiento actualmente en procesamiento.

#### Justificación

Este enfoque evita depender de una cantidad fija de líneas por establecimiento y permite reconstruir registros de longitud variable.

### 2. Reconstrucción de establecimientos multilínea

Para reconstruir los establecimientos se utiliza una variable temporal:

```python
escuela_actual
```

Cuando el parser encuentra un nuevo rango de mesas, el establecimiento anterior se guarda en la lista de resultados y comienza un nuevo registro.

Si una línea no inicia un nuevo establecimiento, su contenido se agrega a `escuela_actual`.

Al finalizar el archivo también se guarda explícitamente el último establecimiento procesado.

### 3. Detección de secciones

La Carta Marina 2023 identifica el comienzo de una nueva sección después del resumen correspondiente a la sección anterior.

El parser utiliza una variable de estado:

```python
esperando_seccion
```

Al detectar:

```text
Resumen Sección
```

el parser queda a la espera del siguiente encabezado válido con estructura:

```text
número - nombre
```

A partir de esa línea se actualizan `seccion_nro` y `seccion_name`.

Este mecanismo permite diferenciar los encabezados de sección de los correspondientes a los circuitos.

### 4. Detección de circuitos

Los circuitos aparecen mediante estructuras del tipo:

```text
1 - SECCIONAL PRIMERA
2 - SECCIONAL SEGUNDA
4 - NUEVA CORDOBA
40A - MENDIOLAZA
```

Por este motivo se utiliza una expresión regular que admite tanto circuitos numéricos como alfanuméricos:

```python
r"^(\d+[A-Z]?)\s*-\s*(.+)$"
```

El número y nombre detectados se conservan como contexto y se asignan posteriormente a cada establecimiento.

### 5. Tipos de mesa

Durante el procesamiento se identificaron distintas formas de indicar el tipo de mesa:

```text
Mixto
Ext Mixto
Extr y Nac. Mixtos
```

En algunos registros correspondientes a `Extr y Nac.`, la palabra `Mixtos` aparece separada en la línea siguiente.

Por este motivo, el parser reconoce esta estructura y normaliza el valor almacenado como:

```text
Extr y Nac. Mixtos
```

La línea independiente que contiene únicamente `Mixtos` se ignora posteriormente para evitar incorporarla al campo `escuela`.

### 6. Conservación del campo escuela

Al igual que en las versiones anteriores, el nombre del establecimiento y su domicilio se conservan dentro de un único campo:

```text
escuela
```

No se intenta separar durante esta etapa:

- nombre del establecimiento;
- dirección;
- barrio;
- localidad.

Esta decisión permite preservar el contenido extraído de la Carta Marina y evita incorporar reglas de normalización dentro del parser.

Las tareas posteriores de normalización de domicilios o geolocalización se realizan de manera independiente sobre los archivos CSV generados.

## Correcciones de inconsistencias de la fuente

Durante la validación se detectaron cinco rangos de mesas cuya extensión resultaba incompatible con la numeración de los establecimientos vecinos y con los resúmenes de los respectivos circuitos.

Los valores extraídos directamente del documento fueron:

| Circuito | Establecimiento | Rango en la fuente | Cantidad resultante |
|----------|-----------------|-------------------:|-------------------:|
| 183 | ESC GRAL NAPOLEON URIBURU | 6152–6227 | 76 |
| 194 | ESC-JUAN B ALBERDI | 6256–6600 | 345 |
| 245 | IPEM N° 55 | 6967–7073 | 107 |
| 361 | ESC.J.M.ESTRADA | 8722–8746 | 25 |
| 364 | ESC.PROV.F.N.LAPRIDA | 8729–8744 | 16 |

El análisis de la continuidad de las mesas, los establecimientos vecinos y los resúmenes de circuito permitió identificar estos rangos como inconsistencias del documento fuente.

Para mantener separada la extracción de las correcciones, el parser primero conserva los valores obtenidos del documento y posteriormente aplica las siguientes modificaciones:

| Circuito | Rango extraído | Rango corregido |
|----------|---------------:|----------------:|
| 183 | 6152–6227 | 6220–6227 |
| 194 | 6256–6600 | 6599–6600 |
| 245 | 6967–7073 | 7066–7073 |
| 361 | 8722–8746 | 8722–8729 |
| 364 | 8729–8744 | 8738–8744 |

Las correcciones modifican únicamente los campos `desde`, `hasta` y `cant_mesas`. El resto de la información extraída del documento se conserva sin modificaciones.

## Validación

Después de aplicar las correcciones se obtienen:

| Concepto | Resultado |
|----------|----------:|
| Establecimientos | 1.586 |
| Mesas contabilizadas | 9.057 |
| Mesas únicas | 9.056 |
| Electores | 3.051.544 |
| Mesa máxima | 9.060 |

No se detectaron registros completamente duplicados en el DataFrame final.

### Mesa duplicada

Permanece duplicado el número de mesa:

```text
4547
```

La duplicación corresponde a dos registros del mismo establecimiento, `IPEM N°172 JOSE HERNANDEZ`, dentro del circuito `88 - TIO PUJIO`.

Uno corresponde al rango ordinario:

```text
4544 - 4549    Mixto
```

y el otro a un registro específico:

```text
4547 - 4547    Extr y Nac. Mixtos
```

Ambos registros se conservan en el dataset debido a que representan categorías diferentes de mesas presentes en la fuente.

### Mesas faltantes

Luego de las correcciones permanecen ausentes de los rangos extraídos los siguientes números de mesa:

```text
6152
6256
6967
8746
```

Estas diferencias se conservan y documentan en lugar de introducir nuevas correcciones que no puedan justificarse directamente a partir de la estructura del documento.

