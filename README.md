# OCR-ImageEnhance
> Optimización de imágenes para el API de "Imagen a Texto" (OCR) de [Computer Vision](https://azure.microsoft.com/en-us/services/cognitive-services/computer-vision/)

Este documento presenta cómo generar una función de Azure de tipo [HTTP Trigger](https://docs.microsoft.com/es-es/azure/azure-functions/functions-triggers-bindings) que recibe la url de una imagen, y la almacena en un blob storage en Azure en escala de grises y con contraste añadido. Un ejemplo de documentos puede ser el que sigue:
>Imagen anterior
<img src="https://github.com/vianeyja/OCR-ImageEnhance/blob/master/imgs/CFE6.jpg" width="200" height="300">

>Imagen posterior a la función
<img src="https://github.com/vianeyja/OCR-ImageEnhance/blob/master/imgs/cfe6bco.jpg" width="200" height="300">





