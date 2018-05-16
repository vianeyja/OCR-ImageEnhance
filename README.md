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

7. Una vez que tenemos creado este parámetro, tenemos que modificar el código de nuestra función HTTP Trigger para obtener la imagen de una url, modificar el color, el contraste, almacenarla en el blob sotrage y regresar la url en donde se almacenó. El código es el siguiente:
```csharp
#r "System.Drawing"
#r "System.IO"
#r "Microsoft.WindowsAzure.Storage"

using System.IO;
using System.Net;
using System.Drawing;
using System.Drawing.Imaging;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.Blob;

public static async Task<HttpResponseMessage> Run(HttpRequestMessage req,  TraceWriter log)
{
    log.Info("C# HTTP trigger function processed a request.");

    //Get the connection string from the Application Settings (The name of the key-value is BroxelString but ypu can change it)
    string connectionstring = GetEnvironmentVariable("BroxelString");

    // url of the image you will convert to black and white
    string url = req.GetQueryNameValuePairs()
        .FirstOrDefault(q => string.Compare(q.Key, "url", true) == 0)
        .Value;

    //Container name where you will store the images
    string containername = req.GetQueryNameValuePairs()
        .FirstOrDefault(q => string.Compare(q.Key, "containername", true) == 0)
        .Value;
    
    string path = req.GetQueryNameValuePairs()
        .FirstOrDefault(q => string.Compare(q.Key, "path", true) == 0)
        .Value;

    if (url == null || containername == null || path == null)
    {
        // Get request body
        dynamic data = await req.Content.ReadAsAsync<object>();
        url = data?.url; 
        containername = data?.containername;
        path = data?.path;
    }

    Bitmap c = (Bitmap)Bitmap.FromStream(new MemoryStream(new WebClient().DownloadData(url))); 
    Bitmap d = MakeGrayscale3(c); 

    var ms = new MemoryStream();
    d.Save(ms, ImageFormat.Png);

    ms.Position = 0;

    // Retrieve storage account from connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(connectionstring);

    // Create the blob client.
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    // Retrieve a reference to a container.
    CloudBlobContainer container = blobClient.GetContainerReference(containername);
    // Create the container if it doesn't already exist.
    await container.CreateIfNotExistsAsync();

    // create a blob in the path of the <container>
    CloudBlockBlob blockBlob = container.GetBlockBlobReference(path);

    await blockBlob.UploadFromStreamAsync(ms);

    
    return url == null
        ? req.CreateResponse(HttpStatusCode.BadRequest, "Please pass a url on the query string or in the request body")
        : req.CreateResponse(HttpStatusCode.OK, blockBlob.Uri);
}

public static Bitmap MakeGrayscale3(Bitmap original)
{

    //create a blank bitmap the same size as original
    Bitmap newBitmap = new Bitmap(original.Width, original.Height);

    //get a graphics object from the new image
    Graphics g = Graphics.FromImage(newBitmap); 

    //create the grayscale ColorMatrix
    ColorMatrix colorMatrix = new ColorMatrix(
            new float[][] {
            new float[] { 0.299f, 0.299f, 0.299f, 0, 0 },
            new float[] { 0.587f, 0.587f, 0.587f, 0, 0 },
            new float[] { 0.114f, 0.114f, 0.114f, 0, 0 },
            new float[] { 0,      0,      0,      1, 0 },
            new float[] { 0,      0,      0,      0, 1 }
    });

    float c = 120 * 0.01f;
    float t = 0.01f;
    ColorMatrix colorMatrixContrast = new ColorMatrix(new float[][] {
            new float[] {c,0,0,0,0},
            new float[] {0,c,0,0,0},
            new float[] {0,0,c,0,0},
            new float[] {0,0,0,1,0},
            new float[] {t,t,t,0,1}
    });
    //create some image attributes
    ImageAttributes attributes = new ImageAttributes();
    ImageAttributes contrast = new ImageAttributes();

    //set the color matrix attribute
    attributes.SetColorMatrix(colorMatrix);
    contrast.SetColorMatrix(colorMatrixContrast);

    //draw the original image on the new image
    //using the grayscale color matrix
    g.DrawImage(original, new Rectangle(0, 0, original.Width, original.Height),
               0, 0, original.Width, original.Height, GraphicsUnit.Pixel, attributes);

            g.DrawImage(newBitmap, new Rectangle(0, 0, original.Width, original.Height),
               0, 0, original.Width, original.Height, GraphicsUnit.Pixel, contrast);

            if (newBitmap.Height > 2000 || newBitmap.Width > 2000)
            {
                newBitmap = new Bitmap(newBitmap, new Size(original.Width / 2, original.Height / 2));
            }
            //dispose the Graphics object
            g.Dispose();
            return newBitmap;
}

public static string GetEnvironmentVariable(string name)
{
    return System.Environment.GetEnvironmentVariable(name, EnvironmentVariableTarget.Process);
}
```
Para consumir esta función, es necesario hacer un llamado de tipo POST a la función. La URL de nuestra función la podemos obtener desde el portal.

<img src="https://github.com/vianeyja/OCR-ImageEnhance/blob/master/imgs/geturl.PNG" width="400" height="80">

El body que espera la función es el siguiente:
* url: la url de la imagen a leer
* containername: el nombre del contenedor donde se almacenarán las imágenes
* path: el path o nombre de donde se almacenará la imagen (ejemplo: imagenblanconegro/imagen.jpg)

<img src="https://github.com/vianeyja/OCR-ImageEnhance/blob/master/imgs/post.PNG" width="400" height="200">

La respuesta que regresa el servicio debe ser con estatus 200 y la url de nuestra imagen en blanco y negro dentro del body.

<img src="https://github.com/vianeyja/OCR-ImageEnhance/blob/master/imgs/response.PNG" width="400" height="200">

Una vez que contamos con la imagen, podemos llamar a nuestro servicio cognitivo o probarlo en el [demo](https://azure.microsoft.com/en-us/services/cognitive-services/computer-vision/) disponible en la página.

<img src="https://github.com/vianeyja/OCR-ImageEnhance/blob/master/imgs/cs.PNG" width="400" height="200">

##Referencias
[]()
