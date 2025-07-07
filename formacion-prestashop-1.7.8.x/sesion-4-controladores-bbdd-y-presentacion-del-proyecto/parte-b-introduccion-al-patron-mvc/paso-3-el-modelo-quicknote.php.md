# Paso 3: El Modelo (Quicknote.php)

Empezaremos por el Modelo, ya que es la base de nuestros datos. Representa una única nota.

**Acción**: Abrir el archivo `modules/quicknotes/classes/Quicknote.php` y pegar el siguiente código, explicando cada parte:

* Las propiedades públicas (`$note_text`, `$active`, etc.).
* La definición estática (`$definition`), que es el mapa que conecta las propiedades con las columnas de la base de datos y sus reglas de validación.

```php
<?php
class Quicknote extends ObjectModel {
    public $id_quicknote;
    public $note_text;
    public $active;
    public $date_add;
    public $date_upd;

    public static $definition = [
        'table' => 'quicknote',
        'primary' => 'id_quicknote',
        'fields' => [
            'note_text' => ['type' => self::TYPE_HTML, 'validate' => 'isCleanHtml', 'required' => true],
            'active'    => ['type' => self::TYPE_BOOL, 'validate' => 'isBool', 'required' => true],
            'date_add'  => ['type' => self::TYPE_DATE, 'validate' => 'isDate'],
            'date_upd'  => ['type' => self::TYPE_DATE, 'validate' => 'isDate'],
        ],
    ];
}
```
