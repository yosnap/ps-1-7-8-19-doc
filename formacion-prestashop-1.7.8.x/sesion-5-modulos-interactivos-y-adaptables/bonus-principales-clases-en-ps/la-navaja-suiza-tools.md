# ## La navaja suiza: "Tools"

La clase `Tools` es la navaja suiza del desarrollador de PrestaShop. Es una clase de "utilidades" que contiene una enorme colección de métodos estáticos (`::`) diseñados para realizar tareas comunes de forma segura, estandarizada y eficiente.

Al ser métodos estáticos, no necesitas crear un objeto (`new Tools()`) para usarlos; simplemente los llamas directamente sobre la clase: `Tools::nombreDelMetodo()`.

***

#### ¿Por qué usar `Tools::` en lugar de funciones nativas de PHP?

1. Seguridad: Métodos como `Tools::getValue()` son más seguros que usar directamente `$_POST` o `$_GET`, ya que aplican filtros y saneamiento básicos.
2. Conveniencia: Simplifican tareas complejas. Por ejemplo, `Tools::displayPrice()` formatea un número con el símbolo de la moneda, los decimales y el formato correctos según la configuración de la tienda, ahorrándote varias líneas de código.
3. Consistencia: Aseguran que ciertas tareas se realicen siempre de la misma manera en todo PrestaShop, manteniendo la coherencia del ecosistema.

***

#### Métodos más Comunes y Útiles de `Tools`

Aquí tienes un desglose de los métodos que usarás con más frecuencia, agrupados por su función.

**1. Gestión de Formularios y Peticiones (Los más usados)**

* `Tools::getValue('nombre_del_campo', 'valor_por_defecto')`
  * Uso: Es la forma segura de obtener un valor de una petición, ya sea `POST` o `GET`. PrestaShop busca la clave primero en `$_GET` y luego en `$_POST`.
  *   Ejemplo:

      PHP

      ```
      // En lugar de: $id_producto = $_POST['id_product'];
      // Usamos:
      $id_product = (int)Tools::getValue('id_product');
      ```
* `Tools::isSubmit('nombre_del_boton_submit')`
  * Uso: Comprueba si un formulario ha sido enviado, verificando si el botón de "Guardar" con un nombre específico está presente en la petición. Es la forma estándar de procesar formularios.
  *   Ejemplo:

      PHP

      ```
      // En el getContent() de un módulo
      if (Tools::isSubmit('submitMiModulo')) {
          // ... El formulario ha sido enviado, procesamos los datos ...
      }
      ```

**2. Redirección y URLs**

* `Tools::redirectAdmin($url)`
  * Uso: Redirige al usuario a una URL del Back Office. Es la forma correcta de hacer redirecciones internas en la administración.
  *   Ejemplo:

      PHP

      ```
      // Redirige al controlador de nuestro módulo
      Tools::redirectAdmin($this->context->link->getAdminLink('AdminQuicknotes'));
      ```
* `Tools::redirect($url)`
  * Uso: Equivalente al anterior, pero para redirigir a URLs del Front Office.

**3. Seguridad**

* `Tools::getToken($seguridadAdicional = true)`
  * Uso: Genera un token de seguridad único para el usuario actual, esencial para proteger los formularios contra ataques CSRF (Cross-Site Request Forgery). Este token se debe añadir como un campo oculto en los formularios y verificarse en el servidor.
  *   Ejemplo:

      PHP

      ```
      // En PHP, para pasarlo a la plantilla
      $this->context->smarty->assign(['mi_token' => Tools::getToken(false)]);

      // En el controlador que recibe los datos
      if (Tools::getToken(false) !== Tools::getValue('token')) {
          die('Token de seguridad inválido');
      }
      ```

**4. Formato y Visualización**

* `Tools::displayPrice($precio, $moneda)`
  * Uso: Formatea un número como un precio, usando la moneda, el símbolo, la posición del símbolo, los decimales y los separadores correctos según la configuración de la tienda y el idioma.
  *   Ejemplo:

      PHP

      ```
      $precio_formateado = Tools::displayPrice(25.5, $this->context->currency);
      // Resultado: "25,50 €" (o "$25.50" dependiendo de la configuración)
      ```
* `Tools::displayDate($fecha)`
  * Uso: Formatea una fecha (`YYYY-MM-DD HH:MM:SS`) a un formato legible según la configuración de idioma de la tienda.
  *   Ejemplo:

      PHP

      ```
      $fecha_formateada = Tools::displayDate('2025-07-02 04:00:00');
      // Resultado: "02/07/2025 04:00:00" (o un formato localizado)
      ```

**5. Depuración (Debug)**

* `Tools::p($variable)` y `Tools::d($variable)`
  * Uso: Son las alternativas de PrestaShop a `print_r` y `var_dump`. Muestran el contenido de una variable de forma legible (envuelta en etiquetas `<pre>`). La diferencia es que `Tools::d()` detiene la ejecución del script después de mostrar la variable (equivalente a `die(var_dump(...))`).
  *   Ejemplo:

      PHP

      ```
      $miArray = ['a' => 1, 'b' => 2];
      Tools::p($miArray); // Muestra el array y el script continúa
      Tools::d($miArray); // Muestra el array y el script se detiene
      ```

En resumen, la clase `Tools` es tu mejor aliada para escribir código seguro, eficiente y consistente con el ecosistema de PrestaShop. Antes de reinventar la rueda o usar una función nativa de PHP, es una buena práctica comprobar si ya existe un método en `Tools` que haga el trabajo por ti.
