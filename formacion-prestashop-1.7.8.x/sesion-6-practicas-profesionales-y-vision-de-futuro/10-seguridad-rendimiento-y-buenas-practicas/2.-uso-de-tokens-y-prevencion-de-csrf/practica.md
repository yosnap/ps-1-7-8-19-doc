# Práctica

La buena noticia es que `HelperForm` gestiona gran parte de esto por nosotros, pero debemos asegurarnos de que la configuración es correcta.

1.  En el método `renderForm()` de `advancedtricks.php`, ya tenemos las líneas clave:

    PHP

    ```php
    // ...
    // Le decimos al helper cuál es nuestro token de seguridad para este contexto
    $helper->token = Tools::getAdminTokenLite('AdminModules');

    // Definimos un nombre único para la acción de envío
    $helper->submit_action = 'submit' . $this->name;
    // ...
    ```

    * `$helper->token`: Asigna el token de seguridad.
    * `$helper->submit_action`: Le da un nombre único al botón de envío.
2. En el método `getContent()`, la comprobación `Tools::isSubmit('submit' . $this->name)` actúa como nuestra primera barrera. Para una seguridad completa, la verificación del token es implícita en el flujo de los controladores de PrestaShop, pero si quisiéramos añadir una capa extra manual, el proceso sería más complejo. Para la configuración de un módulo, confiar en el `HelperForm` y la comprobación `isSubmit` es el estándar establecido y seguro.

**Conclusión del ejercicio:** Debemos entender que al usar `HelperForm` correctamente, la protección básica contra CSRF ya está integrada en el flujo de trabajo de PrestaShop.
