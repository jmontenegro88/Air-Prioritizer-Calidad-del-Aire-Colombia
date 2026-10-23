# Índice de Prioridad de Intervención en Calidad del Aire (IPICA)

**Priorización municipal de la contaminación por material particulado en Colombia (2020–2024)**

Proyecto de la asignatura **Análisis de Datos I** — Maestría en Inteligencia Artificial Aplicada (MIAA), Universidad Icesi.

**Autor:** John Fredy Montenegro — Estudiante MIAA Icesi

---

## 1. Contexto

La contaminación del aire por material particulado (PM2.5 y PM10) es uno de los principales riesgos ambientales para la salud pública. La exposición sostenida a PM2.5 se asocia con enfermedades respiratorias y cardiovasculares y con mortalidad prematura. En Colombia, el Instituto de Hidrología, Meteorología y Estudios Ambientales (IDEAM) consolida las mediciones de las redes de vigilancia de calidad del aire (SVCA).

Frente a una gran cantidad de estaciones y municipios monitoreados y a recursos de gestión limitados, es necesario identificar objetivamente qué municipios requieren mayor atención. Este proyecto transforma millones de mediciones horarias en un **Índice de Prioridad de Intervención en Calidad del Aire (IPICA)** que permite una priorización transparente y reproducible.

## 2. Pregunta SMART

> ¿Qué municipios de Colombia presentan la mayor proporción de días con concentraciones de PM2.5 que superan el límite diario recomendado por la OMS (15 µg/m³) durante el periodo 2020–2024, según los datos oficiales del IDEAM, para priorizar acciones gubernamentales de gestión de la calidad del aire?

## 3. Dataset

- **Fuente:** [Calidad del Aire en Colombia — datos.gov.co (IDEAM)](https://www.datos.gov.co/Ambiente-y-Desarrollo-Sostenible/Calidad-del-Aire-en-Colombia/g4t8-zkc3)
- **Identificador del recurso (Socrata):** `g4t8-zkc3`
- **Volumen total del recurso:** ~32,7 millones de registros (mediciones horarias), periodo 2020–2024, cobertura nacional.
- **Acceso:** descarga directa por API (SODA/Socrata), sin registro.
- **Variables usadas:** estación, municipio, departamento, contaminante (`msfl_code`), concentración estándar (µg/m³), fecha/hora y coordenadas.

### Estrategia de descarga

El recurso está ordenado cronológicamente. Dado que las consultas con filtros del lado del servidor sobre un recurso de este tamaño resultan poco eficientes, la descarga se realiza por **tramos a distintos offsets** (uno por año, 2020 a 2024), seleccionando solo las columnas necesarias. El filtrado a PM2.5 / PM10 y la limpieza se hacen localmente con pandas.

El parámetro `FILAS_POR_ANIO` controla cuántas filas crudas se traen por año (por defecto 500.000, ~2,5 millones en total). El resultado se cachea en `aire_crudo.csv`; si ese archivo existe pero tiene un número de filas distinto al solicitado, se vuelve a descargar automáticamente.

## 4. Umbrales de referencia

| Referencia | Contaminante | Promedio | Límite |
|---|---|---|---|
| OMS 2021 | PM2.5 | 24 h | 15 µg/m³ |
| OMS 2021 | PM2.5 | anual | 5 µg/m³ |
| OMS 2021 | PM10 | 24 h | 45 µg/m³ |
| OMS 2021 | PM10 | anual | 15 µg/m³ |
| Colombia — Res. 2254 de 2017 | PM2.5 | 24 h | 37 µg/m³ |
| Colombia — Res. 2254 de 2017 | PM2.5 | anual | 25 µg/m³ |
| Colombia — Res. 2254 de 2017 | PM10 | 24 h | 75 µg/m³ |
| Colombia — Res. 2254 de 2017 | PM10 | anual | 50 µg/m³ |

## 5. Metodología

1. **Descarga** por tramos anuales desde la API del IDEAM.
2. **Limpieza:** filtrado a PM2.5 y PM10, conversión de tipos, descarte de nulos y valores inválidos (concentraciones negativas o implausibles), extracción de fecha y año.
3. **Agregación diaria** por estación y contaminante (base para comparar contra los límites de 24 horas).
4. **Indicadores por municipio (PM2.5):** % de días que exceden el límite OMS, % de días que exceden la norma colombiana, concentración media diaria, percentil 95 y máximo. Se exige un mínimo de días observados para que un municipio sea comparable.
5. **Construcción del IPICA:** cada componente se normaliza al rango [0, 1] (min–max) y se pondera:

   | Componente | Qué mide | Peso |
   |---|---|---|
   | % días que exceden OMS | exposición de la población | 0,40 |
   | Concentración media | carga crónica | 0,25 |
   | Percentil 95 | severidad de los episodios | 0,20 |
   | % días que exceden norma Colombia | incumplimiento normativo | 0,15 |

   El índice resultante se escala de 0 a 100.
6. **Ranking, visualizaciones y exportación** de resultados.

## 6. Resultados principales

Sobre la muestra analizada (545.327 mediciones de material particulado, 125 municipios, 381 estaciones, 2020–2024):

- **Municipios prioritarios:** encabezan el ranking **Amagá** (IPICA 100) y **Floridablanca** (IPICA 98,2), con una marcada concentración de municipios prioritarios en el **Valle de Aburrá** (Amagá, Sabaneta, Itagüí, Medellín, Girón, La Estrella).
- **Brecha normativa:** Amagá supera el límite diario OMS el 88,96 % de los días observados, pero la norma colombiana solo el 17,98 %. Un municipio puede cumplir la norma nacional y aun así exponer a su población a niveles no recomendados por la OMS.
- **Evolución temporal:** el promedio anual de PM2.5 se mantuvo por encima de la guía OMS (5 µg/m³) y por debajo de la norma colombiana (25 µg/m³) durante todo el periodo, con un mínimo en 2023 (~12,6 µg/m³) y un repunte en 2024 (~15 µg/m³).

Como propuesta de uso se plantea el prototipo conceptual **Air-Prioritizer Dashboard**, una herramienta visual para consultar los indicadores y apoyar la focalización de recursos de monitoreo e intervención.

## 7. Estructura del repositorio

```
.
├── IPICA_calidad_del_aire_Colombia.ipynb   # Notebook principal (Google Colab)
├── ipica_ranking_municipios.csv            # Ranking exportado (se genera al ejecutar)
├── aire_crudo.csv                          # Caché de datos crudos (se genera al ejecutar)
└── README.md
```

> `aire_crudo.csv` e `ipica_ranking_municipios.csv` se generan al ejecutar el notebook y no se versionan.

## 8. Cómo ejecutar

1. Abrir `IPICA_calidad_del_aire_Colombia.ipynb` en [Google Colab](https://colab.research.google.com/).
2. Ejecutar todas las celdas (`Entorno de ejecución → Ejecutar todo`).
3. Para forzar una nueva descarga (por ejemplo, tras cambiar `FILAS_POR_ANIO` o los offsets por año), eliminar `aire_crudo.csv` antes de ejecutar.

Dependencias: `requests`, `numpy`, `pandas`, `matplotlib` (todas disponibles por defecto en Colab).

## 9. Limitaciones

- Se trabajó con una muestra por tramos del recurso; ampliarla aumenta la representatividad.
- No se integraron variables meteorológicas, de población ni de fuentes de emisión; el índice es de priorización por exposición, no un modelo causal.
- La cobertura de estaciones de monitoreo no es homogénea entre regiones del país.

## 10. Fuentes

- IDEAM — Calidad del Aire en Colombia (datos.gov.co, recurso `g4t8-zkc3`).
- Organización Mundial de la Salud — Guías de calidad del aire 2021.
- Ministerio de Ambiente y Desarrollo Sostenible — Resolución 2254 de 2017.
