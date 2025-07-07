# Paso 4: El Controlador (AdminQuicknotesController.php)

Ahora crearemos el cerebro que gestionará la página del Back Office.

**Acción**: Abrimos `modules/quicknotes/controllers/admin/AdminQuicknotesController.php` y agregamos el código. Puntos clave del `__construct`:

* `$table` y `$className`: Cómo conectan el controlador con el `ObjectModel`.
* `$fields_list`: Cómo esta "receta" le dice a `HelperList` qué columnas mostrar.
* `renderForm()`: Cómo esta función define los campos para `HelperForm`.

```php
<?php
class AdminQuicknotesController extends ModuleAdminController {
    public function __construct() {
        $this->bootstrap = true;
        $this->table = 'quicknote';
        $this->className = 'Quicknote';
        $this->lang = false;
        parent::__construct();
        $this->addRowAction('edit');
        $this->addRowAction('delete');
        $this->fields_list = [
            'id_quicknote' => ['title' => 'ID', 'align' => 'center', 'class' => 'fixed-width-xs'],
            'note_text' => ['title' => 'Nota', 'filter_key' => 'a!note_text'],
            'active' => ['title' => 'Activo', 'active' => 'status', 'type' => 'bool', 'class' => 'fixed-width-xs'],
            'date_add' => ['title' => 'Fecha', 'type' => 'datetime'],
        ];
    }

    public function renderForm() {
        $this->fields_form = [
            'legend' => ['title' => $this->l('Nota Rápida'), 'icon' => 'icon-pencil'],
            'input' => [
                ['type' => 'textarea', 'label' => $this->l('Texto de la nota'), 'name' => 'note_text', 'autoload_rte' => true, 'required' => true],
                ['type' => 'switch', 'label' => $this->l('Activo'), 'name' => 'active', 'is_bool' => true,
                    'values' => [['id' => 'active_on', 'value' => 1, 'label' => $this->l('Sí')],['id' => 'active_off', 'value' => 0, 'label' => $this->l('No')]],
                ],
            ],
            'submit' => ['title' => $this->l('Guardar'), 'class' => 'btn btn-default pull-right']
        ];
        return parent::renderForm();
    }
}
```

{% hint style="info" %}
Nota: Aún no hemos creado la tabla ni el menú. Eso lo haremos en el siguiente paso.
{% endhint %}
