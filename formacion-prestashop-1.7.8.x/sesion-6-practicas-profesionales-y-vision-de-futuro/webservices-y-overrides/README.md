# Webservices y Overrides

### **1. Webservices**

* Explicación: Una API que permite a aplicaciones externas leer/escribir datos de forma segura.
* Ejercicio Práctico: Activa el Webservice desde Parámetros Avanzados > Webservice. Crea una clave de API con permisos para leer `products`. Accede desde el navegador a `http://<tu-tienda>.com/api/products/?ws_key=<tu_clave_api>` para ver el listado de productos en formato XML.

### **2. Overrides**

* Explicación: Técnica avanzada para modificar el comportamiento del núcleo sin editar sus archivos. Debe usarse como último recurso.
*   Ejercicio Práctico: Crea un override para la clase `Address.php` (`modules/advancedtricks/override/classes/Address.php`) para que el campo "teléfono" sea siempre obligatorio, incluso si la configuración del país no lo requiere.

    PHP

    ```
    <?php
    class Address extends AddressCore {
        public static function getValidationRules($className = __CLASS__) {
            // 1. Obtenemos las reglas de validación originales
            $rules = parent::getValidationRules($className);

            // 2. Modificamos la regla para el campo 'phone'
            $rules['required']['phone'] = true;

            // 3. Devolvemos las reglas modificadas
            return $rules;
        }
    }
    ```

    * Instrucción: Recerda que se puede borrar el archivo `/var/cache/class_index.php` para que PrestaShop detecte el override.
