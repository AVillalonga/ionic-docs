---
previousText: 'Guardando fotos en el sistema de archivos'
previousUrl: '/docs/angular/your-first-app/3-loading-photos'
nextText: 'Añadiendo soporte móvil'
nextUrl: '/docs/angular/your-first-app/5-deploying-mobile'
---

# Cargando fotos desde el sistema de archivos

Hemos implementado la toma de fotos y su guardado en el sistema de archivos. Falta una última pieza de funcionalidad: las fotos se almacenan en el sistema de archivos, pero necesitamos una forma de guardar punteros en cada archivo para que puedan mostrarse nuevamente en la galería de fotos.

Afortunadamente, esto es fácil: aprovecharemos la [API Storage ](https://capacitor.ionicframework.com/docs/apis/storage) de Capacitor para almacenar nuestro arreglo de fotos en un formato clave-valor.

## API Almacenamiento

Comience definiendo una variable constante que actuará como la clave para el almacenamiento de las fotografias:

```typescript
export class PhotoService {
 public photos: Photo[] = [];
 private PHOTO_STORAGE: string = "photos";

  // otro código
}
```

Luego, al final de la funcion `addNewToGallery`, hará una llamada a `Storage.set()` para guardar el arreglo de fotografias "photos". Al agregarlo aquí, la matriz de Fotos se almacena cada vez que se toma una nueva foto. De esta forma, no importa cuando el usuario, cierre la app o se cambie a una app distinta todas las fotografias estan guardadas.

```typescript
Storage.set({
  key: this.PHOTO_STORAGE,
  value: JSON.stringify(this.photos)
});
```

Con los datos del arreglo de fotos guardados, crearemos una función llamada `loadSaved()` que pueda recuperar esos datos. Utilizamos la misma clave para recuperar el array de fotos en formato JSON, luego analizarlo en una matriz:

```typescript
public async loadSaved() {
  // Retrieve cached photo array data
  const photoList = await Storage.get({ key: this.PHOTO_STORAGE });
  this.photos = JSON.parse(photoList.value) || [];

  // more to come...
}
```

On mobile (coming up next!), we can directly set the source of an image tag - `<img src="x" />` - to each photo file on the Filesystem, displaying them automatically. On the web, however, we must read each image from the Filesystem into base64 format, using a new `base64` property on the `Photo` object. This is because the Filesystem API uses [IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API) under the hood. Below is the code you need to add in the `loadSaved()` function you just added:

```typescript
// Display the photo by reading into base64 format
for (let photo of this.photos) {
  // Read each saved photo's data from the Filesystem
  const readFile = await Filesystem.readFile({
      path: photo.filepath,
      directory: FilesystemDirectory.Data
  });

  // Web platform only: Load the photo as base64 data
  photo.webviewPath = `data:image/jpeg;base64,${readFile.data}`;
}
```

Después, llama a este nuevo método en `tab2.page. s` de modo que cuando el usuario navega por primera vez a Tab 2 (la Galería de fotos), todas las fotos se cargan y se muestran en la pantalla.

```typescript
async ngOnInit() {
  await this.photoService.loadSaved();
}
```

¡Eso es! Hemos construido una función completa de Galería de Fotos en nuestra aplicación Ionic que funciona en la web. ¡Próximamente, lo transformaremos en una aplicación móvil para iOS y Android!
