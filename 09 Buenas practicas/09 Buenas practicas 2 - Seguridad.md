# 9.2 Seguridad

https://angular.io/guide/security

# 2.1 Actualizar la aplicación

Actualizar regualrmente Angular para ir aplicando las correccioens de seguridad que vayan publicando.

Usar `npm audit` para detectar vulnerabilidades en las dependencias y poder mantenerlas actualizadas.

<br>

# 2.2 No modificar la copia de Angular

Angular incluye esta recomendación, asi que supongo que hay quien lo hace.

<br>

# 2.3 Evitar `security risk`

En la documentación de Angular se marcan los elementos que tienen un riesgo de seguridad, con la etiqueta `security risk`

Por ejemplo, https://angular.io/api/core/ElementRef

<br>

# 2.4 XSS (Cross-Site scripting)

Angular trata todos los valores como no confiables
- Cuando se inserta un valor en un template usando template binding o interpolation, Angular lo sanea automáticamente.

- Si un valor ha sido saneado ya fuera de Angular hay que informar a Angular que es un valor seguro, usando el servicio `DomSanitizer`.

- Angular considera los templates confiables por defecto y se tratan como código ejecutable.
  - Nunca generar plantillas concatenando información introducida por el usuario.
  - El uso de AOT en producción previene esto porque precompila las plantillas.

<br>

> Hay que usar siempre `template binding` (property / attribute binding )
  <br> o `string interpolation`, para mostrar información en una plantilla.

<br>

## 2.4.1 DomSanitizer
---

Angular nos ofrece un servicio para marcar algo como confiable.
  - `bypassSecurityTrustHtml`
  - `bypassSecurityTrustScript`
  - `bypassSecurityTrustStyle`
  - `bypassSecurityTrustUrl`
  - `bypassSecurityTrustResourceUrl`

https://angular.io/api/platform-browser/DomSanitizer

<br>

# 2.5 CSP (Content-Security-Policy)

- Se configura en el servidor web y se envía al cliente mediante un header.

- Indica al navegador las fuentes de contenido confiables, para que solo ejecute o muestre recursos de estas fuentes.

  ```
   `Content-Security-Policy: script-src 'self' https://apis.google.com`
  ```

  Solo ejecuta secuencias de comandos del dominio actual y de `apis.google.com`

- Se puede aplicar a varios recursos: script-src, frame-src,  font-src, image-src, style-src, ...

- `default-src` cambia el comportamiento predeterminado de todos los recursos. Por defecto es same-origin

- Se puede establecer una política para una página específica, definiendo un meta

  ```html
  <meta http-equiv="Content-Security-Policy"
    content="default-src https://cdn.example.net; child-src 'none'; object-src 'none'">`
  ```

- CSP solo se aplica a Javascript existente en ficheros, que se cargan como recursos.
  - No aplicaría a JS incluido directamente en etiquetas `<script>`, controladores en linea, URL 'javacript:...'
  - Para asegurarse CSP rechaza completamente las secuencias de comando integradas.
  - Rechaza el uso de `eval`
  - Es una buena práctica no usar JS integrado, aunque no usemos CSP

# 2.6 CSRF/XSRF (Cross-site Request Forgery)

Es una técnica para asegurarnos de las peticiones provienen del origen esperado.

Se basa en el siguiente proceso
  - el servidor envia un token en una cookie (por defecto `XSRF-TOKEN`)
  - el cliente lee la cookie
  - el cliente envía una cabecera (X-XSRF-TOKEN) con el valor de la cookie, en todas las peticiones mutables (en POST si, en GET no)
  - El servidor compara la cabecera con el valor guardado en sesión y rechaza la petición si no coincide.

Si un atacante usara un enlace para acceder a una web desde donde invocar al api de nuestra aplicación, no podrá acceder a la cookie de nuestro dominio, y por tanto, no podrá ejecutar peticiones en el API.

<br>

El servicio `HttpClient` soporta la parte de cliente, hay que configurar el server para que envie la cookie.

https://angular.io/api/common/http/HttpClientXsrfModule

```ts
imports: [
  HttpClientModule,
  HttpClientXsrfModule.withOptions({
    cookieName: 'My-Xsrf-Cookie',
    headerName: 'My-Xsrf-Header',
  }),
],
```
<br>
