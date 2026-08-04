# carta-marina-cordoba

Proyecto para la extracción, procesamiento y análisis de datos de las Cartas Marinas de Córdoba (2015–2023).

---

# Parser 2015

## Repositorio de referencia

Este parser se desarrolló tomando como base el trabajo realizado por **Open Data Córdoba**, disponible en el siguiente repositorio:

[OpenDataCordoba/elecciones2015/](https://github.com/OpenDataCordoba/elecciones2015/)

En particular, se utilizó como referencia el parser correspondiente a la extracción de la **Carta Marina 2015** incluido en dicho proyecto.

Este trabajo no busca reemplazar el repositorio original, sino comprender su funcionamiento, reproducir su lógica y documentar las modificaciones necesarias para adaptarlo a un entorno moderno de desarrollo.

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

---

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

---

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

---

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

---

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

---

## Resultado

El parser mantiene la estructura general del algoritmo original, introduciendo únicamente las modificaciones necesarias para:

- Ejecutarse correctamente en Python 3.
- Funcionar en un entorno moderno como Google Colab.
- Procesar correctamente las particularidades detectadas en la Carta Marina 2015 sin intervención manual durante la extracción.

Las tareas de limpieza y normalización de los datos se realizan en una etapa posterior, preservando el parser como un proceso dedicado exclusivamente a la extracción de información.
