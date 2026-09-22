# Patrones Invisibles: Información, Incertidumbre y Movilidad Urbana

Proyecto Retador — ML No Supervisado, Universidad de La Sabana, 2026-II.

Consultoría analítica simulada para **UrbanMove**: aplicamos teoría de la información, clustering por
partición y modelos de mezcla gaussiana sobre 1,458,644 viajes de taxi en Nueva York para responder
dónde y cuándo posicionar conductores, minimizando el tiempo de espera sin sobre-ofertar en zonas de
baja demanda.

## Equipo

| Integrante | Código | Rol |
|---|---|---|
| Diego Alejandro Sandoval | 339271 | Rol 4 — Estratega de Negocio y Selección de Datos |
| Daniel Figueredo | 332679 | Rol 1 — Analista de Incertidumbre e Información |
| Juan Esteban Ocampo | 286388 | Rol 2 — Arquitecto de Segmentación por Partición |
| Juan Andres Martinez | 326771 | Rol 3 — Modelador de Mezclas y Variables Latentes |

## Documento de soporte

El PDF con los resultados completos, la justificación de la metodología y las recomendaciones
cuantificadas está en [`docs/documento_soporte.pdf`](docs/documento_soporte.pdf).

## Dataset

[NYC Taxi Trip Duration (Kaggle)](https://www.kaggle.com/c/nyc-taxi-trip-duration/data) — no se
incluye en el repositorio (ver [`data/instrucciones_dataset.md`](data/instrucciones_dataset.md) para reproducir).

## Notebooks (correr en este orden)

| Notebook | Contenido |
|---|---|
| [`notebooks/00_limpieza_y_muestreo.ipynb`](notebooks/00_limpieza_y_muestreo.ipynb) | Preparación de datos común a los 3 roles |
| [`notebooks/01_rol1_teoria_informacion.ipynb`](notebooks/01_rol1_teoria_informacion.ipynb) | Rol 1 — entropía, KL, información mutua, log-sum-exp |
| [`notebooks/02_rol2_clustering.ipynb`](notebooks/02_rol2_clustering.ipynb) | Rol 2 — K-means y DBSCAN |
| [`notebooks/03_rol3_mezclas_gaussianas.ipynb`](notebooks/03_rol3_mezclas_gaussianas.ipynb) | Rol 3 — GMM, EM implementado a mano |
| [`notebooks/04_rol4_integracion_recomendaciones.ipynb`](notebooks/04_rol4_integracion_recomendaciones.ipynb) | Rol 4 — integración y recomendaciones |

`src/config.py` contiene las constantes compartidas (semilla, proyección geográfica, tamaño de zona)
que usan los notebooks 01 a 04.

## Cómo correrlo

```bash
pip install pandas numpy scipy scikit-learn pyarrow matplotlib
```

1. Descargar el dataset (ver `data/instrucciones_dataset.md`).
2. Correr los notebooks en el orden numerado (00 → 04). Cada uno guarda los archivos `.parquet`
   que el siguiente necesita.

## Metodología (resumen)

- **Limpieza:** se eliminan solo errores de medición físicamente imposibles (1.35% del dataset). Los
  días atípicos reales (ventisca de enero, Memorial Day) se conservan deliberadamente.
- **Muestreo:** estratificado proporcional por día×hora, 50,004 viajes, semilla 42 — verificado por
  divergencia KL contra la población.
- **Rol 1:** los agregados de teoría de la información se calculan sobre la población completa
  (1,438,943 viajes) para evitar el sesgo de estimación detectado en la muestra.
- **Rol 3:** el algoritmo EM se implementó también a mano (sin scikit-learn) y se verificó que
  reproduce exactamente los mismos resultados, con criterio de convergencia estricto (1e-9).

El detalle completo de cada decisión — con sus alternativas consideradas y su justificación — está en
[`bitacora_metodologica.md`](bitacora_metodologica.md).
