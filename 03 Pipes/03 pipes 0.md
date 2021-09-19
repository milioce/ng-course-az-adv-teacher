# Custom Pipes

27. Creating a custom pipe
28. Pipes as providers

## Create new project

```bash
$ ng generate application pipes -s -t --style=scss --routing=true
```
<br>

Add script to start the application

`package.json`
```json
    "start:pipes": "ng serve --project pipes",
```
<br>

## Initial Files

- `demo/demo-pipe.component.ts`
- `demo/demo-provider.component.ts`
- `demo/file.ts`
- `demo/filesize.pipe.ts`
- `app/module.ts`
- `app/app.component.ts`
- `src/styles.scss`

<br>

## DemoPipeComponent

`demo/demo-pipe.component.ts`
``` ts
import { Component, OnInit } from '@angular/core';
import { File } from './file';

@Component({
  selector: 'demo-pipe',
  template: `
    <div>
      <div *ngFor="let file of files">
        <p>{{ file.name }}</p>
        <p>{{ file.size }}</p>
      </div>
    </div>
  `
})

export class DemoPipeComponent implements OnInit {
  files: File[];

  constructor() { }

  ngOnInit() {
    this.files = [
      { name: 'logo.svg', size: 2120109, type: 'image/svg' },
      { name: 'banner.jpg', size: 18029, type: 'image/jpg' },
      { name: 'background.png', size: 1784562, type: 'image/png' }
    ];
   }
}
```
<br>

## DemoProviderComponent

`demo/demo-provider.component.ts`
``` ts
import { Component, OnInit } from '@angular/core';
import { File } from './file';

@Component({
  selector: 'demo-provider',
  template: `
    <div>
      <div *ngFor="let file of files">
        <p>{{ file.name }}</p>
        <p>{{ file.size }}</p>
      </div>
    </div>
  `
})

export class DemoProviderComponent implements OnInit {
  files: File[];

  constructor() { }

  ngOnInit() {
    this.files = [
      { name: 'logo.svg', size: 2120109, type: 'image/svg' },
      { name: 'banner.jpg', size: 18029, type: 'image/jpg' },
      { name: 'background.png', size: 1784562, type: 'image/png' }
    ];
   }
}
```
<br>

## File

`demo/file.ts`
``` ts
export interface File {
  name: string,
  size: number,
  type: string
}
```
<br>


## FileSizePipe

`demo/filesize.pipe.ts`
``` ts
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'filesize'
})

export class FileSizePipe implements PipeTransform {
  transform(value: any, ...args: any[]): any {
  }
}
```
<br>


## AppModule

`app/module.ts`
``` ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { DemoPipeComponent } from './demo/demo-pipe.component';
import { DemoProviderComponent } from './demo/demo-provider.component';

@NgModule({
  declarations: [
    AppComponent,
    DemoPipeComponent,
    DemoProviderComponent
    FileSizePipe
  ],
  imports: [
    BrowserModule,
    AppRoutingModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```
<br>


## AppComponent

`app/app.component.ts`
``` ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <demo-pipe></demo-pipe>
  `,
  styles: []
})
export class AppComponent {
  title = 'pipes';
}
```
<br>
