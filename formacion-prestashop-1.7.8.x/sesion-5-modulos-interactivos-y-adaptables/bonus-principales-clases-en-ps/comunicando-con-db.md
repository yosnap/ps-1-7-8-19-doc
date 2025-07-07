# ## Comunicando con Db

La clase `Db` es el puente de comunicación entre tu código PHP y la base de datos de la tienda. Es la herramienta central y estandarizada de PrestaShop para realizar cualquier tipo de consulta SQL (leer, insertar, actualizar o borrar datos).

***

#### El Punto de Partida: `Db::getInstance()`

Para usar la clase `Db`, casi siempre empezarás con `Db::getInstance()`. Este método estático te devuelve el objeto de conexión activo a la base de datos. Piensa en él como "dame acceso a la base de datos ahora mismo".

PHP

```
$db = Db::getInstance();
```

Una vez que tienes este objeto, puedes usar sus métodos para ejecutar consultas.

***

#### Métodos Principales para Ejecutar Consultas

**1. `->execute($sql)`**

* Uso: Para consultas que no devuelven resultados, como `INSERT`, `UPDATE`, `DELETE`, o `CREATE TABLE`.
* Resultado: Devuelve `true` si la consulta se ejecutó con éxito y `false` si falló.
*   Ejemplo:

    PHP

    ```
    $sql = "DELETE FROM `" . _DB_PREFIX_ . "quicknote` WHERE id_quicknote = 5";
    $exito = Db::getInstance()->execute($sql);
    if ($exito) {
        // La nota se borró correctamente
    }
    ```

**2. `->executeS($sql)`**

La "S" es de "Select".

* Uso: Para consultas `SELECT` que pueden devolver múltiples filas.
* Resultado: Devuelve un `array` de `arrays`. Cada `array` interior representa una fila de resultados. Si no hay resultados, devuelve un `array` vacío.
*   Ejemplo:

    PHP

    ```
    $sql = "SELECT id_product, name FROM `" . _DB_PREFIX_ . "product_lang` WHERE id_lang = 1 LIMIT 5";
    $productos = Db::getInstance()->executeS($sql);
    // $productos ahora es un array como:
    // [
    //   ['id_product' => 1, 'name' => 'Camiseta...'],
    //   ['id_product' => 2, 'name' => 'Taza...'],
    //   ...
    // ]
    foreach ($productos as $producto) {
        echo $producto['name'];
    }
    ```

**3. `->getRow($sql)`**

* Uso: Para consultas `SELECT` de las que esperas obtener una sola fila.
* Resultado: Devuelve un único `array` asociativo que representa esa fila. Si no hay resultados, devuelve `false`.
*   Ejemplo:

    PHP

    ```
    $sql = "SELECT * FROM `" . _DB_PREFIX_ . "customer` WHERE id_customer = 10";
    $cliente = Db::getInstance()->getRow($sql);
    // $cliente ahora es un array como: ['id_customer' => 10, 'firstname' => 'John', ...]
    if ($cliente) {
        echo $cliente['firstname'];
    }
    ```

**4. `->getValue($sql)`**

* Uso: Para consultas `SELECT` de las que solo necesitas un único valor (el primer campo de la primera fila). Es muy eficiente para obtener un `COUNT(*)` o un solo campo.
* Resultado: Devuelve el valor directamente (un string, un número, etc.). Si no hay resultados, devuelve `false`.
*   Ejemplo:

    PHP

    ```
    $sql = "SELECT COUNT(*) FROM `" . _DB_PREFIX_ . "product` WHERE active = 1";
    $numero_productos = Db::getInstance()->getValue($sql);
    // $numero_productos ahora contiene el número total, ej: 152
    ```

***

#### ¡Importante! Seguridad y Prevención de Inyección SQL

Nunca debes insertar variables directamente en una cadena SQL sin limpiarlas primero. La clase `Db` te ayuda a hacerlo de forma segura.

* Para números: Siempre convierte la variable a su tipo correcto. Esto se llama "casteo" o "type casting". Es la forma más segura.
  * Correcto: `$sql = "SELECT * FROM tabla WHERE id = " . (int)$id_variable;`
  * Incorrecto: `$sql = "SELECT * FROM tabla WHERE id = " . $id_variable;`
* Para texto (strings): Usa la función `pSQL()`. Esta función escapa caracteres especiales que podrían ser usados para un ataque de inyección SQL.
  * Correcto: `$sql = "SELECT * FROM tabla WHERE nombre = '" . pSQL($nombre_variable) . "'";`
  * Incorrecto: `$sql = "SELECT * FROM tabla WHERE nombre = '" . $nombre_variable . "'";`

En resumen, la clase `Db` es la herramienta central para cualquier operación con la base de datos. Usar sus métodos `executeS`, `getRow` y `getValue` te ahorra código, y utilizar siempre `pSQL()` o el casteo de tipos (`(int)`) en tus variables es absolutamente obligatorio para proteger tu tienda.
