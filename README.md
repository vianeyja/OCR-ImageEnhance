# OCR-ImageEnhance
> Optimización de imágenes para el API de "Imagen a Texto" (OCR) de [Computer Vision](https://azure.microsoft.com/en-us/services/cognitive-services/computer-vision/)

Este documento presenta cómo generar una función de Azure de tipo [HTTP Trigger](https://docs.microsoft.com/es-es/azure/azure-functions/functions-triggers-bindings) que recibe la url de una imagen, y la almacena en un blob storage en Azure en escala de grises y con contraste añadido. Un ejemplo de documentos puede ser el que sigue:
>Imagen anterior
<img src="https://github.com/vianeyja/OCR-ImageEnhance/blob/master/imgs/CFE6.jpg" width="200" height="300">

>Imagen posterior a la función
<img src="https://github.com/vianeyja/OCR-ImageEnhance/blob/master/imgs/cfe6bco.jpg" width="200" height="300">

Este trataimento a la imagen, permite que la lectura del servicio cognitivo mejore de manera considerable.
Los pasos a seguir son los siguientes:

1. Crear una cuenta de almacenamiento en Azure. Puedes encontrar las instrucciones en la siguiente [liga](https://docs.microsoft.com/es-es/azure/storage/common/storage-create-storage-account)
2. Generar un contenedor de blobs dentro de la cuenta que acabamos de crear. Puedes encontrar las instrucciones en la siguiente [liga](https://docs.microsoft.com/es-es/azure/vs-azure-tools-storage-explorer-blobs)
3. Generar una Function App, para contener nuestra función.
<img src="https://github.com/vianeyja/OCR-ImageEnhance/blob/master/imgs/FunctionApp.PNG" width="400" height="300">
4. Agregar una función de tipo "HTTP Trigger", la cual va a funcionar cada vez que reciba una llamada de tipo POST.
> Lo primero que debemos de tener en cuenta en esta función, es agregar una configuración de la aplicación para almacenar nuestra cadena de conexión al blob Storage.
5. Una vez creada nuestra función, navegar a ella, y dar clic en "App Settings".
<img src="https://github.com/vianeyja/OCR-ImageEnhance/blob/master/imgs/AppSettings.PNG" width="400" height="200">

6. Navegar a "Application Settings". Agregar uno nuevo y configurar el nombre que queramos para la cadena de conexión y el valor.
(Puedes encontrar la cadena de conexión a tu cuenta de storage en el portal de Azure, navegando a los settings > keys)

<img src="https://github.com/vianeyja/OCR-ImageEnhance/blob/master/imgs/AppSettings2.PNG" width="200" height="200">




