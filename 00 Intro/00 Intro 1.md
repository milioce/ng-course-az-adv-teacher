# 1. Crear Worspace multi proyecto

## 1.1 Crear workspace

```bash
$ ng new curso-angular-avanzado --createApplication=false --interactive=false
```
<br>

El fichero `angular.json` tiene solamente la configuración del workspace

```json
{
  "$schema": "./node_modules/@angular/cli/lib/config/schema.json",
  "version": 1,
  "newProjectRoot": "projects",
  "projects": {
  }
}
```

## 1.2 Crear aplicacion

Creamos la aplicación "components"

```bash
ng generate application components -t -s --style=scss --routing=true
```

Crea los ficheros para la aplicación en la carpeta `projects/components`
- 2 ficheros `tsconfig.*.ts` que extienden la configuración del workspace `tsconfig.ts`
- La carpeta `src` con el código de la aplicación
- El fichero `karma.conf.js` con la configuración de tests
- La carpeta `e2e` para los tests de protractor.

<br>

El primer proyecto que creemos será el proyecto por defecto

También añade un proyecto de tipo aplicación al fichero `angular.json`

```json
{
  "$schema": "./node_modules/@angular/cli/lib/config/schema.json",
  "version": 1,
  "newProjectRoot": "projects",
  "projects": {
    "components": {
      "projectType": "application",
      "schematics": {
        "@schematics/angular:component": {
          "style": "scss"
        }
      },
      "root": "projects/components",
      "sourceRoot": "projects/components/src",
      "prefix": "app",
      "architect": {
        "build": {
          "builder": "@angular-devkit/build-angular:browser",
          "options": {
            "outputPath": "dist/components",
            "index": "projects/components/src/index.html",
            "main": "projects/components/src/main.ts",
            "polyfills": "projects/components/src/polyfills.ts",
            "tsConfig": "projects/components/tsconfig.app.json",
            "aot": true,
            "assets": [
              "projects/components/src/favicon.ico",
              "projects/components/src/assets"
            ],
            "styles": [
              "projects/components/src/styles.scss"
            ],
            "scripts": []
          },
          "configurations": {
            "production": {
              "fileReplacements": [
                {
                  "replace": "projects/components/src/environments/environment.ts",
                  "with": "projects/components/src/environments/environment.prod.ts"
                }
              ],
              "optimization": true,
              "outputHashing": "all",
              "sourceMap": false,
              "namedChunks": false,
              "extractLicenses": true,
              "vendorChunk": false,
              "buildOptimizer": true,
              "budgets": [
                {
                  "type": "initial",
                  "maximumWarning": "2mb",
                  "maximumError": "5mb"
                },
                {
                  "type": "anyComponentStyle",
                  "maximumWarning": "6kb",
                  "maximumError": "10kb"
                }
              ]
            }
          }
        },
        "serve": {
          [...]
        },
        "extract-i18n": {
          [...]
        },
        "test": {
          [...]
        },
        "lint": {
          [...]
        },
        "e2e": {
          [...]
        }
      }
    }
  },
  "defaultProject": "components"
}
```
<br>

## 1.3 Lanzar una aplicación

<br>

Especificamos el proyecto que deseamos lanzar

``` bash
ng serve --project=components
```
<br>


Si no especificamos un proyecto, se lanza el proyecto por defecto

``` bash
ng serve
```
<br>

Podemos lanzar varias aplicaciones a la vez, si las configuramos en puertos distintos.

<br>

# 2. Crear mock. Json-server

https://github.com/typicode/json-server

<br>

## 2.1 Instalar json server

Se puede instalar globalmente o en un proyecto.

``` bash
$ npm install -g json-server
```
<br>

## 2.2 Crear el fichero `db.json` con los datos

``` json
{
  "products": [
    { "id": 1, "price": 2800, "name": "MacBook Pro" },
    { "id": 2, "price": 50, "name": "USB-C Adaptor" },
    { "id": 3, "price": 400, "name": "iPod" },
    { "id": 4, "price": 900, "name": "iPhone" },
    { "id": 5, "price": 600, "name": "Apple Watch" }
  ]
}
```
<br>

## 2.3 Lanzar JSON Server

``` bash
$ json-server --watch db.json
```
<br>

Por defecto arranca el servidor en http://localhost:3000

## 2.4 Rutas

Por defecto crea rutas basadas en el fichero `db.json`.
<br> Un resource para cada key (productos)

- GET    /products
- GET    /products/1
- POST   /products
- PUT    /products/1
- PATCH  /products/1
- DELETE /products/1

> Definir alias para rutas

Se crea un fichero `routes.json`

``` json
{
  "/api/*": "/$1",
  "/articles/:id": "/products/:id",
  "/articles?id=:id": "/products/:id"
}
```
<br>

``` bash
$ json-server --watch db.json --routes routes.json
```
<br>

# 3. Angular proxy

## 3.1 Definir un proxy en Angular
---
Vamos a usar un proxy en angular para poder usar `http://localhost:4200/api` y que nos redirija a `http:/localhost:3000`

- creamos el fichero `proxy.conf.json`en el raiz

`proxy.conf.json`
```json
{
  "/api": {
    "target": "http://localhost:3000",
    "secure": false
  }
}
```
<br>

Añadimos un comando al `package.json`
```json
"start:proxy": "ng serve --proxy-config proxy.conf.json",
```

Probamos el comando ejecutando
```bash
npm run start:proxy
```

## 3.2 Ejecutar tareas concurrentemente

Instalamos el paquete `concurrently`
```bash
npm install concurrently --save-dev
```

Añadimos un comando al `package.json`
```json
"start:mock": "concurrently \"npm run mock:server\" \"npm run start:proxy\"",
```

Ejecutamos el comando

```bash
npm run start:mock
```
