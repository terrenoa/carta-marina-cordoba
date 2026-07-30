# carta-marina-cordoba
Proyecto para la extracción, procesamiento y análisis de datos de las Cartas Marinas de Córdoba (2015–2023)
# 2015

## Diferencias respecto del repositorio original

Este proyecto busca reproducir el funcionamiento del parser original de la Carta Marina, realizando únicamente las adaptaciones necesarias para ejecutarlo en un entorno moderno (Google Colab / Python 3) y corregir incompatibilidades detectadas durante el procesamiento.

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

`splitlines()` elimina correctamente los caracteres de salto de página (`\f`) generados por `pdftotext`, evitando líneas vacías y caracteres residuales sin modificar el contenido del documento.

---

## 2. Validación de continuidad de mesas

### Original

Ante una discontinuidad en la numeración de mesas, el parser finaliza inmediatamente mediante:

```python
exit(1)
```

### Adaptación

Se reemplazó por:

```python
raise ValueError(...)
```

### Justificación

Esta modificación permite identificar con precisión la línea que produjo el error y facilita la depuración dentro de Google Colab, sin alterar la lógica de validación.

---

## 3. Manejo de saltos de página entre circuitos

### Problema detectado

En la Carta Marina 2015 existen circuitos cuyo encabezado aparece al final de una página y cuyo listado de establecimientos continúa en la página siguiente.

El parser original considera que una línea vacía finaliza el bloque de escuelas. En estos casos, la primera escuela del circuito queda omitida.

Ejemplo:

```
Circuito 4D

(Página siguiente)

COL VILLA EUCARISTICA...
```

### Adaptación

Las líneas vacías pasan a ignorarse sin abandonar el estado de lectura de escuelas.

El estado únicamente finaliza al encontrar:

```
Resúmen del Circuito
```

### Justificación

Esta modificación permite atravesar correctamente los saltos de página sin perder registros.

---

## 4. Reconstrucción de nombres de establecimientos

### Problema detectado

Algunas direcciones contienen separaciones de cuatro espacios utilizadas también como delimitador de columnas.

Ejemplo:

```
ESC PANAMERICANA - SOTO 919    - B°ACOSTA    9    00337 a 00345    3.078
```

El parser original interpreta esta línea como cinco columnas en lugar de cuatro.

### Adaptación

En lugar de asumir que el nombre del establecimiento ocupa una única columna, se reconstruye utilizando todos los elementos previos a las tres últimas columnas.

Conceptualmente:

- Última columna → Cantidad de electores.
- Penúltima → Rango de mesas.
- Antepenúltima → Cantidad de mesas.
- Todo lo anterior → Nombre y dirección del establecimiento.

### Justificación

El README del repositorio original ya advierte que las direcciones presentan inconsistencias y pueden requerir correcciones manuales.

Esta adaptación automatiza uno de esos casos sin modificar la estructura general del parser.
