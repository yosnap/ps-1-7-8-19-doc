# Práctica: Creación del Módulo advancedtricks

Objetivo: Tener un módulo funcional con una página de configuración que contenga un campo de texto (`textarea`). Sobre esta base, aplicaremos los ejercicios de seguridad y rendimiento.

**Paso 1: Estructura de Archivos**

Crea la siguiente estructura de carpetas y archivos vacíos:

```
/modules
  └── /advancedtricks/
      ├── config.xml
      ├── advancedtricks.php
      └── /views/
          └── /templates/
              └── /hook/
                  └── display.tpl
```

**Paso 2: Código del `config.xml`**

XML

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<module>
    <name>advancedtricks</name>
    <displayName><![CDATA[Trucos Avanzados]]></displayName>
    <version><![CDATA[1.0.0]]></version>
    <description><![CDATA[Módulo para practicar seguridad, rendimiento y overrides.]]></description>
    <author><![CDATA[Tu Nombre]]></author>
    <is_configurable>1</is_configurable>
</module>
```

**Paso 3: Código del `advancedtricks.php` (versión base)**

Este código crea la página de configuración con el formulario, pero todavía sin la lógica de validación de seguridad (que añadiremos en el ejercicio).

PHP

```php
<?php
if (!defined('_PS_VERSION_')) exit;

class AdvancedTricks extends Module
{
    const ADVANCEDTRICKS_TEXT_KEY = 'ADVANCEDTRICKS_TEXT';

    public function __construct() {
        $this->name = 'advancedtricks';
        $this->version = '1.0.0';
        $this->author = 'Tu Nombre';
        $this->bootstrap = true;
        parent::__construct();
        $this->displayName = $this->l('Trucos Avanzados');
        $this->description = $this->l('Módulo para practicar seguridad y rendimiento.');
    }

    public function install() {
        return parent::install() && $this->registerHook('displayHome');
    }

    public function uninstall() {
        return parent::uninstall() && Configuration::deleteByName(self::ADVANCEDTRICKS_TEXT_KEY);
    }

    public function getContent() {
        $output = '';
        if (Tools::isSubmit('submit' . $this->name)) {
            // AQUÍ IRÁ EL CÓDIGO DEL EJERCICIO DE VALIDACIÓN
            // De momento, solo guardamos el valor
            $texto_usuario = Tools::getValue(self::ADVANCEDTRICKS_TEXT_KEY);
            Configuration::updateValue(self::ADVANCEDTRICKS_TEXT_KEY, $texto_usuario, true);
            $output .= $this->displayConfirmation($this->l('Ajuste guardado'));
        }
        return $output . $this->renderForm();
    }

    public function renderForm() {
        $fields_form[0]['form'] = [
            'legend' => ['title' => $this->l('Configuración de Seguridad')],
            'input' => [
                [
                    'type' => 'textarea',
                    'label' => $this->l('Mi Texto (permite HTML)'),
                    'name' => self::ADVANCEDTRICKS_TEXT_KEY,
                    'autoload_rte' => true, // Activa el editor de texto enriquecido
                ],
            ],
            'submit' => ['title' => $this->l('Guardar')]
        ];

        $helper = new HelperForm();
        $helper->module = $this;
        $helper->name_controller = $this->name;
        $helper->token = Tools::getAdminTokenLite('AdminModules');
        $helper->currentIndex = AdminController::$currentIndex . '&configure=' . $this->name;
        $helper->submit_action = 'submit' . $this->name;
        $helper->fields_value[self::ADVANCEDTRICKS_TEXT_KEY] = Configuration::get(self::ADVANCEDTRICKS_TEXT_KEY);
        return $helper->generateForm($form_fields);
    }

    public function hookDisplayHome($params) {
        $this->context->smarty->assign([
            'mi_texto_guardado' => Configuration::get(self::ADVANCEDTRICKS_TEXT_KEY)
        ]);
        return $this->display(__FILE__, 'views/templates/hook/display.tpl');
    }
}
```

**Paso 4: Código de la Plantilla (`display.tpl`)**

Crea el archivo `views/templates/hook/display.tpl`. Aquí es donde mostraremos el texto guardado.

Fragmento de código

```
{* Este es el archivo donde haremos el ejercicio de "escapado" *}
```

***

#### Ahora, Aplicamos el Ejercicio de "Validación y Prevención de XSS"

Con la base del módulo ya creada, ahora puedes explicar a tus alumnos cómo modificarlo para añadir la capa de seguridad.

1\. Backend (Validación): En el archivo `advancedtricks.php`, dentro del método `getContent()`, reemplaza el bloque `if (Tools::isSubmit(...))` con el siguiente código, que incluye la validación:

PHP

```php
// En el método getContent()
if (Tools::isSubmit('submit' . $this->name)) {
    // Obtenemos el valor del campo
    $texto_usuario = Tools::getValue(self::ADVANCEDTRICKS_TEXT_KEY);

    // --- INICIO DEL CÓDIGO DEL EJERCICIO ---
    // Validamos si el HTML es limpio antes de guardar
    if (!Validate::isCleanHtml($texto_usuario)) {
        $output .= $this->displayError($this->l('El texto contiene código no permitido.'));
    } else {
        // Si es seguro, lo guardamos y mostramos confirmación
        Configuration::updateValue(self::ADVANCEDTRICKS_TEXT_KEY, $texto_usuario, true); // El 'true' permite guardar HTML
        $output .= $this->displayConfirmation($this->l('Ajuste guardado correctamente.'));
    }
    // --- FIN DEL CÓDIGO DEL EJERCICIO ---
}
```

2\. Frontend (Escapado): En el archivo `views/templates/hook/display.tpl`, añade este código para mostrar el resultado de forma segura:

Fragmento de código

```smarty
<div style="border:1px solid orange; padding: 15px; margin: 15px 0;">
    <h3>Contenido Guardado:</h3>

    <h4>Opción 1: Segura por defecto (escapando HTML)</h4>
    <p>Si mostramos el texto con el filtro de seguridad 'escape', PrestaShop convierte las etiquetas HTML en texto plano para que no se ejecuten.</p>
    <pre><code>{$mi_texto_guardado|escape:'htmlall':'UTF-8'}</code></pre>
    <p><b>Resultado:</b></p>
    <div style="border: 1px dashed #ccc; padding: 10px;">
        {$mi_texto_guardado|escape:'htmlall':'UTF-8'}
    </div>
    <hr>
    
    <h4>Opción 2: Mostrar HTML (solo si ha sido validado)</h4>
    <p>Usamos 'nofilter' porque confiamos en nuestra validación 'isCleanHtml' del backend. Esto renderizará el HTML seguro que permitimos.</p>
    <pre><code>{$mi_texto_guardado nofilter}</code></pre>
    <p><b>Resultado:</b></p>
    <div style="border: 1px dashed #ccc; padding: 10px;">
        {$mi_texto_guardado nofilter}
    </div>
</div>
```
