# 1. Validación y Prevención de XSS

* Explicación: La regla de oro: nunca confíes en la entrada del usuario. Debemos validar los datos en el backend antes de guardarlos y "escapar" los datos en el frontend antes de mostrarlos para prevenir ataques de Cross-Site Scripting (XSS).
* Ejercicio Práctico:
  1. Crearemos un formulario en el `getContent()` del módulo `advancedtricks` (caso práctico) con un campo de texto (`textarea`).
  2.  Backend (Validación): En `postProcess()`, al guardar, usa `Validate::isCleanHtml()` para limpiar el texto de cualquier script malicioso.

      PHP

      ```php
      // En postProcess()
      $texto_usuario = Tools::getValue('mi_texto');
      if (!Validate::isCleanHtml($texto_usuario)) {
          $this->errors[] = 'El texto contiene código no permitido.';
      } else {
          Configuration::updateValue('ADVANCEDTRICKS_TEXT', $texto_usuario, true); // El 'true' permite guardar HTML
          // ...
      }
      ```
  3.  Frontend (Escapado): En un hook, muestra este texto en una plantilla `.tpl` usando el modificador de Smarty `|escape`. Explica que `nofilter` solo se puede usar si confías 100% en que el dato ha sido limpiado previamente (como hicimos en el paso anterior).

      Fragmento de código

      ```
      {* Opción segura por defecto para cualquier texto *}
      <p>{$mi_texto_guardado|escape:'htmlall':'UTF-8'}</p>

      {* Opción para mostrar HTML, solo si ha sido validado con isCleanHtml *}
      <div>{$mi_texto_guardado nofilter}</div>
      ```
