# Carta Marina Córdoba

Proyecto de extracción y estructuración de datos de las **Cartas Marinas de la Provincia de Córdoba** correspondientes a las elecciones provinciales de 2013, 2015, 2017, 2019 y 2023.

El objetivo es transformar los documentos originales, publicados en formato PDF, en datasets estructurados y comparables que permitan posteriormente realizar tareas de normalización, geolocalización y análisis espacial de los establecimientos de votación.

## Origen del proyecto

El trabajo toma como punto de partida desarrollos previos de **Open Data Córdoba**, particularmente los repositorios correspondientes a las elecciones provinciales de 2015 y a la Carta Marina 2017.

A partir de esos antecedentes se adaptaron y desarrollaron parsers para las distintas ediciones de la Carta Marina, manteniendo un modelo de datos común y documentando las diferencias entre los formatos publicados en cada elección.

## Flujo de trabajo

El proyecto se organiza en etapas:

```text
Cartas Marinas PDF
        ↓
Conversión a TXT
        ↓
Parsing y validación
        ↓
CSV estructurados
        ↓
Normalización / geolocalización
        ↓
Análisis y visualización
