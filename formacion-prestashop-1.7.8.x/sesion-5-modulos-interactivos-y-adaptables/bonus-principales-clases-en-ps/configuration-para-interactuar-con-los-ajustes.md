# ## Configuration - Para interactuar con los ajustes

La clase `Configuration` es el gestor central de ajustes. Te permite guardar, leer y eliminar de forma segura los pares "clave-valor" en la tabla `ps_configuration` de la base de datos.

***

#### 1. `updateValue()`

Este es el método principal para guardar o actualizar un valor de configuración.

* Propósito: Crea una nueva entrada en la tabla de configuración si la clave no existe, o actualiza el valor si ya existe.
*   Firma:

    PHP

    ```php
    public static function updateValue(string $key, mixed $values, bool $html = false, ?int $id_shop_group = null, ?int $id_shop = null): bool
    ```
* Parámetros Clave:
  * `$key`: El nombre único de tu ajuste (ej. `'MI_MODULO_COLOR'`).
  * `$values`: El valor que quieres guardar. Puede ser un texto, un número o un array (PrestaShop lo serializará automáticamente).
  * `$html`: `true` si el valor contiene HTML. Esto le dice a PrestaShop que no debe limpiar las etiquetas HTML al guardar. Úsalo con precaución.
  * `$id_shop`: Si proporcionas un ID de tienda, el ajuste se guardará solo para esa tienda. Si es `null`, se guarda globalmente o para el grupo de tiendas.
*   Ejemplo de Uso:

    PHP

    ```php
    // Guardar un valor global para todas las tiendas
    Configuration::updateValue('MI_MODULO_API_KEY', 'xyz123abc');

    // Guardar un ajuste específico para la tienda actual
    $id_tienda_actual = $this->context->shop->id;
    Configuration::updateValue('MI_MODULO_ACTIVADO', 1, false, null, $id_tienda_actual);
    ```

***

#### 2. `get()`

El método principal para leer o recuperar un valor de configuración.

* Propósito: Obtiene el valor asociado a una clave, respetando el contexto multitienda.
*   Firma:

    PHP

    ```php
    public static function get(string $key, ?int $id_lang = null, ?int $id_shop_group = null, ?int $id_shop = null, $default = false)
    ```
* Parámetros Clave:
  * `$key`: El nombre del ajuste que quieres leer.
  * `$id_shop`: Si proporcionas un ID de tienda, buscará primero el valor para esa tienda. Si no lo encuentra, buscará el del grupo de tiendas y, finalmente, el valor global.
  * `$default`: El valor que devolverá la función si la clave no se encuentra en la base de datos. Muy útil para evitar errores.
*   Ejemplo de Uso:

    PHP

    ```php
    // Leer una clave global
    $apiKey = Configuration::get('MI_MODULO_API_KEY');

    // Leer una clave específica de la tienda actual, y si no existe, devolver '0'
    $id_tienda_actual = $this->context->shop->id;
    $estaActivado = Configuration::get('MI_MODULO_ACTIVADO', null, null, $id_tienda_actual, '0');
    ```

***

#### 3. `deleteByName()`

El método principal para eliminar un valor de configuración.

* Propósito: Elimina una clave de configuración y todos sus valores asociados (globales, por tienda o por grupo). Es esencial usarlo en el método `uninstall()` de tu módulo para no dejar basura en la base de datos.
*   Firma:

    PHP

    ```
    public static function deleteByName(string $key): bool
    ```
*   Ejemplo de Uso:

    PHP

    ```php
    // En el método uninstall() de tu módulo
    public function uninstall()
    {
        Configuration::deleteByName('MI_MODULO_API_KEY');
        Configuration::deleteByName('MI_MODULO_ACTIVADO');
        return parent::uninstall();
    }
    ```

***

#### 4. `getMultiple()`

Un método de utilidad para leer varios valores a la vez de forma eficiente.

* Propósito: En lugar de hacer múltiples llamadas a `Configuration::get()`, le pasas un array de claves y te devuelve todos sus valores.
*   Firma:

    PHP

    ```
    public static function getMultiple(array $keys, ?int $id_lang = null, ?int $id_shop_group = null, ?int $id_shop = null): array
    ```
*   Ejemplo de Uso:

    PHP

    ```php
    $claves = ['MI_MODULO_API_KEY', 'MI_MODULO_ACTIVADO', 'PS_SHOP_NAME'];
    $configuraciones = Configuration::getMultiple($claves);

    // $configuraciones ahora es un array asociativo:
    // [
    //   'MI_MODULO_API_KEY' => 'xyz123abc',
    //   'MI_MODULO_ACTIVADO' => '1',
    //   'PS_SHOP_NAME' => 'Mi Tienda'
    // ]
    echo $configuraciones['PS_SHOP_NAME'];
    ```

***

#### 5. `hasContext()`

Un método de comprobación muy útil en entornos multitienda.

* Propósito: Verifica si un valor de configuración ha sido definido específicamente para un contexto de tienda o grupo de tiendas, o si simplemente está heredando el valor global.
*   Firma:

    PHP

    ```php
    public static function hasContext(string $key, ?int $id_shop = null, ?int $id_shop_group = null): bool
    ```
*   Ejemplo de Uso:

    PHP

    ```php
    // Comprobar si la tienda 2 tiene su propio valor para MI_MODULO_ACTIVADO
    if (Configuration::hasContext('MI_MODULO_ACTIVADO', 2)) {
        // Sí, la tienda 2 tiene un ajuste personalizado.
    } else {
        // No, está usando el ajuste global o del grupo.
    }php
    ```
