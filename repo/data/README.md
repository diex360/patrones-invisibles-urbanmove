# Datos

Este proyecto usa **NYC Taxi Trip Duration** (Kaggle):
https://www.kaggle.com/c/nyc-taxi-trip-duration/data

El archivo `train.csv` no se incluye en el repositorio por su tamaño (~65 MB) y por la licencia de Kaggle.

## Para reproducir

1. Crear una cuenta gratuita en Kaggle si no tienes una.
2. Descargar `nyc-taxi-trip-duration.zip` desde el enlace de arriba.
3. Descomprimir y colocar `train.csv` en esta carpeta (`data/train.csv`).
4. Correr los notebooks en orden, empezando por `00_limpieza_y_muestreo.ipynb`, que genera:
   - `data/train_limpio.parquet` (población limpia, usada por el Rol 1)
   - `data/muestra_features.parquet` (muestra de 50,004 viajes con variables derivadas, usada por los Roles 2 y 3)
   - `data/poblacion_zonas.parquet` (población con el mismo grid de zonas, para validar el Rol 1)

Estos `.parquet` tampoco se versionan (ver `.gitignore`); cada notebook los regenera o los consume desde el paso anterior.
