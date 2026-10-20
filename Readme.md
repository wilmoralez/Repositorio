# Preprocesamiento de Imágenes Sentinel-2 para Clustering K-TAU (Valle de Sula)

Este repositorio contiene el código en R Markdown utilizado para cargar, recortar y estructurar datos satelitales multiespectrales de la misión Sentinel-2. El objetivo de este script es aislar la región del Valle de Sula (Honduras) y transformar los datos espaciales en una matriz plana para su posterior análisis con el algoritmo de clustering robusto K-TAU.

**Autor:** Wilmer Adonay Morales Cantarero  
**Fecha:** Octubre 2026  

## 📦 Descarga de Datos (Importante)

Debido a las restricciones de tamaño de GitHub, la imagen satelital original no se encuentra alojada en este repositorio. Para ejecutar el código, se debe descargar la carpeta `.SAFE` desde Google Drive.

1. Descargue los datos satelitales desde el siguiente enlace:
   👉 https://drive.google.com/drive/folders/1pIeX5zR_eSjZSuZjgPj_CdWPsXsexMc5?usp=sharing
2. Descomprima el archivo (si aplica) y asegúrese de que la carpeta resultante se llame exactamente:
   `S2A_MSIL2A_20201112T161531_N0500_R140_T16PCB_20230303T113427.SAFE`
3. Coloque esta carpeta en el **mismo directorio** donde tiene guardado el archivo R Markdown (`.Rmd`).

## 🛠️ Requisitos Previos

El script está desarrollado en el entorno estadístico R. Necesitará tener instalado el paquete `terra` para el manejo de datos espaciales. Puede instalarlo ejecutando el siguiente comando en su consola de R:

```R
install.packages("terra")