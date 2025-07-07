# Tema #6 Configuraciones desde el Back Office

### **1. Uso de `HelperForm`: El constructor de formularios**

`HelperForm` es la herramienta "clásica" de PrestaShop para generar formularios en el Back Office que sean consistentes con el estilo de la plataforma.

* Explicación: Se basa en definir la estructura del formulario mediante un array de PHP. Es rápido y potente para configuraciones de módulos.
* Ejercicio: En el controlador del Back Office de nuestro módulo, crea un formulario simple con un solo campo de texto para guardar un "Título personalizado".

```php
// En el método renderForm() del controlador
private function renderForm() {
        $fields_form[0]['form'] = [
            'legend' => [
                'title' => $this->l('Ajustes Básicos'),
                'icon' => 'icon-cogs',
            ],
            'input' => [
                [
                    'type' => 'text',
                    'label' => $this->l('Título del Banner'),
                    'name' => 'CONFIG_AVANZADA_TITULO',
                    'required' => true,
                ],
            ],
            'submit' => [
                'title' => $this->l('Guardar'),
            ],
        ];

        $helper = new HelperForm();
        $helper->module = $this;
        $helper->name_controller = $this->name;
        $helper->token = Tools::getAdminTokenLite('AdminModules');
        $helper->currentIndex = AdminController::$currentIndex . '&configure=' . $this->name;
        $helper->submit_action = 'submit' . $this->name;

        // Precargar valor existente
        $helper->fields_value['CONFIG_AVANZADA_TITULO'] = Configuration::get('CONFIG_AVANZADA_TITULO');

        return $helper->generateForm($fields_form);
    }
```

### **2. Guardar y Precargar Datos con `Configuration`**

Un formulario debe mostrar los valores ya guardados y persistir los nuevos cambios.

* Explicación: Se usan dos funciones clave:
  * `Configuration::updateValue('MI_CLAVE', 'mi_valor')`: Guarda o actualiza un valor en la tabla `ps_configuration`.
  * `Configuration::get('MI_CLAVE')`: Recupera un valor guardado.
* Ejercicio:
  1. En el método `postProcess()` del controlador, valida y guarda el valor del campo de texto usando `Configuration::updateValue()`.
  2. En el método `renderForm()`, usa `Configuration::get()` para rellenar el campo del formulario con el valor que ya está en la base de datos.

```php
    public function getContent() {
        $titulo = Tools::getValue('CONFIG_AVANZADA_TITULO');
        if ($titulo) {
            Configuration::updateValue('CONFIG_AVANZADA_TITULO', $titulo);
            return $this->displayConfirmation($this->l('Guardado')) . $this->renderForm();
        }

        // Solo mostrar formulario
        return $this->renderForm();
    }
```

### **3. `ps_configuration` vs. Base de Datos Personalizada**

* Explicación:
  * `ps_configuration`: Úsala para ajustes sencillos, globales y de tipo "clave-valor" (ej: un color, un texto, activar/desactivar una opción).
  * Tabla Personalizada: Úsala cuando necesites almacenar múltiples registros de datos estructurados (ej: un historial de cambios, una lista de elementos, valoraciones de productos).

### **4. Introducción a FormBuilder de Symfony**

* Explicación: Para las páginas completamente nuevas del Back Office (no la configuración de un módulo), PrestaShop 1.7+ utiliza el constructor de formularios de Symfony, que es más moderno y potente. Funciona creando una clase `Type` que define el formulario.
* Ejercicio (Conceptual): Muestra la estructura básica de cómo se vería una acción de un controlador de Symfony, explicando que es la dirección hacia la que se mueve PrestaShop, aunque `HelperForm` sigue siendo vital.

```php
<?php

if (!defined('_PS_VERSION_')) {
    exit;
}

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\Extension\Core\Type\SubmitType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\HttpFoundation\Request;

// Clase del formulario inline
class SimpleFormType extends AbstractType {
    public function buildForm(FormBuilderInterface $builder, array $options) {
        $builder
            ->add('titulo', TextType::class, [
                'label' => 'Título del Banner',
                'required' => true,
                'data' => $options['data']['titulo'] ?? '', // Precargar datos
            ])
            ->add('save', SubmitType::class, [
                'label' => 'Guardar',
                'attr' => ['class' => 'btn btn-primary']
            ]);
    }
}

class ConfiguracionAvanzada extends Module {

    public function __construct() {
        $this->name = 'configuracionavanzada';
        $this->tab = 'others';
        $this->version = '1.0.0';
        $this->author = 'Desarrollo PS';
        $this->need_instance = 0;
        $this->bootstrap = true;

        parent::__construct();

        $this->displayName = $this->l('Configuración Avanzada (Symfony)');
        $this->description = $this->l('Módulo con FormBuilder de Symfony.');
        $this->ps_versions_compliancy = ['min' => '1.7.7', 'max' => _PS_VERSION_];
    }

    public function install() {
        return parent::install() &&
            Configuration::updateValue('CONFIG_AVANZADA_TITULO', $this->l('Título por defecto'));
    }

    public function uninstall() {
        return parent::uninstall() &&
            Configuration::deleteByName('CONFIG_AVANZADA_TITULO');
    }

    /**
     * Página de configuración con FormBuilder
     */
    public function getContent() {
        // Obtener valor actual
        $valorActual = Configuration::get('CONFIG_AVANZADA_TITULO', '');

        try {
            // Crear Request manualmente desde $_POST y $_GET
            $request = Request::createFromGlobals();

            // Crear formulario con datos actuales
            $form = $this->get('form.factory')->create(SimpleFormType::class, [
                'titulo' => $valorActual
            ]);

            // Procesar petición
            $form->handleRequest($request);

            $output = '';

            // Verificar si se envió y es válido
            if ($form->isSubmitted() && $form->isValid()) {
                $data = $form->getData();
                Configuration::updateValue('CONFIG_AVANZADA_TITULO', $data['titulo']);
                $output = $this->displayConfirmation($this->l('Guardado correctamente'));

                // Actualizar valor para mostrar en el formulario
                $valorActual = $data['titulo'];
            } elseif ($form->isSubmitted()) {
                $output = $this->displayError($this->l('Error en el formulario'));
            }

            // Renderizar formulario
            return $output . $this->renderSymfonyForm($form);
        } catch (Exception $e) {
            // Fallback si Symfony no está disponible
            return $this->renderFallbackForm();
        }
    }

    /**
     * Renderizar formulario con Symfony
     */
    private function renderSymfonyForm($form) {
        $formView = $form->createView();
        $valorActual = Configuration::get('CONFIG_AVANZADA_TITULO', '');

        $html = '
        <div class="panel">
            <div class="panel-heading">
                <i class="icon-cogs"></i> ' . $this->l('Configuración con Symfony FormBuilder') . '
            </div>
            <div class="panel-body">
                <div class="alert alert-info">
                    <strong>Valor actual:</strong> ' . htmlspecialchars($valorActual) . '
                </div>';

        // Usar el formulario de Symfony directamente
        $html .= '<form method="post" action="' . htmlspecialchars($_SERVER['REQUEST_URI']) . '">';

        // Iterar sobre los campos del formulario
        foreach ($formView->children as $child) {
            if ($child->vars['name'] === 'titulo') {
                $html .= '
                    <div class="form-group">
                        <label class="control-label required">' . htmlspecialchars($child->vars['label']) . '</label>
                        <input type="text"
                               name="' . $formView->vars['full_name'] . '[titulo]"
                               value="' . htmlspecialchars($valorActual) . '"
                               class="form-control"
                               required>
                        <p class="help-block">' . $this->l('Introduce el título del banner') . '</p>
                    </div>';
            }
        }

        // Token CSRF si existe
        if (isset($formView->children['_token'])) {
            $html .= '<input type="hidden" name="' . $formView->vars['full_name'] . '[_token]" value="' . $formView->children['_token']->vars['value'] . '">';
        }

        $html .= '
                <div class="panel-footer">
                    <button type="submit" class="btn btn-default pull-right">
                        <i class="process-icon-save"></i> ' . $this->l('Guardar') . '
                    </button>
                </div>
            </form>
            </div>
        </div>';

        return $html;
    }

    /**
     * Formulario de respaldo si Symfony falla
     */
    private function renderFallbackForm() {
        $valorActual = Configuration::get('CONFIG_AVANZADA_TITULO', '');

        // Procesar formulario simple
        if (Tools::isSubmit('submitFallback')) {
            $titulo = Tools::getValue('CONFIG_AVANZADA_TITULO');
            if (!empty($titulo)) {
                Configuration::updateValue('CONFIG_AVANZADA_TITULO', $titulo);
                $output = $this->displayConfirmation($this->l('Guardado correctamente'));
                $valorActual = $titulo;
            } else {
                $output = $this->displayError($this->l('El título no puede estar vacío'));
            }
        } else {
            $output = '';
        }

        $html = '
        <div class="panel">
            <div class="panel-heading">
                <i class="icon-cogs"></i> ' . $this->l('Configuración (Modo Fallback)') . '
            </div>
            <div class="panel-body">
                <div class="alert alert-warning">
                    <strong>Modo de compatibilidad:</strong> Symfony FormBuilder no disponible
                </div>
                <div class="alert alert-info">
                    <strong>Valor actual:</strong> ' . htmlspecialchars($valorActual) . '
                </div>

                <form method="post" action="' . htmlspecialchars($_SERVER['REQUEST_URI']) . '">
                    <div class="form-group">
                        <label class="control-label required">' . $this->l('Título del Banner') . '</label>
                        <input type="text"
                               name="CONFIG_AVANZADA_TITULO"
                               value="' . htmlspecialchars($valorActual) . '"
                               class="form-control"
                               required>
                        <p class="help-block">' . $this->l('Introduce el título del banner') . '</p>
                    </div>

                    <div class="panel-footer">
                        <button type="submit" name="submitFallback" class="btn btn-default pull-right">
                            <i class="process-icon-save"></i> ' . $this->l('Guardar') . '
                        </button>
                    </div>
                </form>
            </div>
        </div>';

        return $output . $html;
    }
}

```

```php
// Ejemplo conceptual, no para implementar ahora
public function miAccion(Request $request): Response
{
    $form = $this->createForm(MiFormularioType::class);
    $form->handleRequest($request);

    if ($form->isSubmitted() && $form->isValid()) {
        // ... Lógica para guardar
        return $this->redirectToRoute('ruta_de_exito');
    }

    return $this->render('@Modules/configuracionavanzada/templates/admin/mi_pagina.html.twig', [
        'form' => $form->createView(),
    ]);
}
```
