# Custom Pipes

27. Creating a custom pipe
28. Pipes as providers

## 1. Creating a custom pipe

Vamos a crear un pipe para transformar el tamaño del fichero en MB.

Creamos un fichero `app/filesize.pipe.ts`

`app/demo/filesize.pipe.ts`
``` ts
import { Pipe, PipeTransform } from "@angular/core"

@Pipe({
  name: 'filesize'
})
export class FileSizePipe implements PipeTransform {
  transform(value) {
    console.log(value);
  }
}
```
<br>

> El pipe tiene que devolver el valor transformado.
> - El primer argumento es el valor a transformar
> - El resto son parámetros adicionales del pipe

`app/filesize.pipe.ts`
``` ts
import { Pipe, PipeTransform } from "@angular/core"

@Pipe({
  name: 'filesize'
})
export class FileSizePipe implements PipeTransform {
  transform(size: number, extension: string = 'MB') {
    return (size / (1024 * 1024)).toFixed(2) + extension;
  }
}
```
<br>

> Especificamos el argumento en la plantilla

`app/demo/demo-pipe.component.ts`
```html
        <p>{{ file.size | filesize:' megabytes' }}</p>
```
<br>


# 2. Pipes as providers

Vamos a usar el pipe en el componente en lugar de en la plantilla.

## 2.1 Activar `demo-provider`

`app/app.component.ts`
```html
  template: `
    <demo-provider></demo-provider>
  `,
```
<br>


## 2.2 Definimos un provider en el componente

Lo definimos como provider en el componente y lo inyectamos en el constructor.

`app/demo/demo-provider.component.ts`
```ts
@Component({
  selector: 'demo-provider',
  template: `
    <div>
      <div *ngFor="let file of files">
        <p>{{ file.name }}</p>
        <p>{{ file.size }}</p>
      </div>
    </div>
  `,
 + providers: [ FileSizePipe ]
})

export class DemoProviderComponent implements OnInit {
  files: File[];

+ constructor(private fileSizepipe: FileSizePipe) { }
}
```
<br>

> Creamos un array transformando los datos de `files`.
> - Utilizamos el método `transform`

`app/demo/demo-provider.component.ts`
```ts
  ngOnInit() {
+   const files = [
      { name: 'logo.svg', size: 2120109, type: 'image/svg' },
      { name: 'banner.jpg', size: 18029, type: 'image/jpg' },
      { name: 'background.png', size: 1784562, type: 'image/png' }
    ];

    this.files = files.map(file => {
      return {
        name: file.name,
        type: file.type,
        size: this.fileSizepipe.transform(file.size, 'mb')
      }
    });
   }
```
<br>
