# 9.3 Accesibility

# 3.1 Accesibility attributes

```html
  <button [attr.aria-label]="myActionLabel">...</button>
  <button aria-label="Save document">...</button>`
```
<br>

# 3.2 Angular Material

- Es una librería mantenida por el equipo de Angular, que promete ser accesible.

- El CDK incluye alguna herramienta para accesibilidad
  - `LiveAnnouncer`: anunciar mensajes en lectores de pantalla utilizando un aria-live

  - `cdkTrapFocus`: Directiva para capturar el foco de la tecla TAB dentro de un elemento.
  <br> Se usa por ejemplo en los modales, donde el foco debe estar restringido.

<br>

# 3.3 Usar elementos nativos.

- Los elementos nativos de HTML ya incluyen una interacción estandar implementada.

- Es mejor aplicar directivas a elementos nativos, que construir componentes custom e implementar toda la interacción.

- Usar contenedores para elementos nativos.
  - Si utilizamos un input en nuestro componente custom, quien use nuestro componente no le podrá añadir propiedades o atributos.

  - Si creamos un contenedor que proyecte el input desde fuera del componente  con `ng-content`, si que se podriá.
  <br><br>Por ejemplo, `MatFormField`

  ```html
    <mat-form-field appearance="legacy">
      <mat-label>Legacy form field</mat-label>
      <input matInput placeholder="Placeholder">
      <mat-icon matSuffix>sentiment_very_satisfied</mat-icon>
      <mat-hint>Hint</mat-hint>
    </mat-form-field>
  ```
  <br>