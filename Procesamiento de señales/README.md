# Procesamiento de señales

Apuntes de teoría y prácticas en Python organizados por fecha de clase.

## Organización

```text
teoria/                  Apuntes de cada clase en Markdown
material_teorico/        PDFs, presentaciones y otros archivos de teoría
notebooks/               Código de cada clase en notebooks compatibles con Colab
plantillas/              Modelos para crear una nueva clase
datos/                   Archivos de datos utilizados en las prácticas
```

## Índice de clases

Agregar una fila por clase, usando la fecha real y enlaces a sus archivos.

| Fecha | Tema | Teoría | Notebook |
| --- | --- | --- | --- |

## Agregar una clase

1. Copiar `plantillas/clase.md` a `teoria/AAAA-MM-DD.md`.
2. Copiar `plantillas/clase.ipynb` a `notebooks/AAAA-MM-DD.ipynb`.
3. Reemplazar `AAAA-MM-DD` por la fecha de la clase (por ejemplo, `2026-10-28`) y completar el tema y el contenido.
4. Agregar los enlaces de ambos archivos al índice de arriba.

Usar el mismo nombre de fecha para la teoría y el notebook de una clase permite encontrarlos fácilmente. Si hubo dos clases el mismo día, agregar un sufijo en ambos archivos, por ejemplo `AAAA-MM-DD-clase-1`.

## Usar los notebooks en Google Colab

En Google Colab, elegir **Archivo → Subir notebook** y seleccionar el archivo `.ipynb` de la carpeta `notebooks/`. También se pueden abrir desde Google Drive si se sube la carpeta allí.

Al terminar, descargar el notebook como `.ipynb` y guardar la versión actualizada en esta carpeta. Los cambios hechos en Colab no se sincronizan automáticamente con el archivo local.

Los archivos de `datos/` deben subirse también a la sesión de Colab cuando una práctica los necesite.
