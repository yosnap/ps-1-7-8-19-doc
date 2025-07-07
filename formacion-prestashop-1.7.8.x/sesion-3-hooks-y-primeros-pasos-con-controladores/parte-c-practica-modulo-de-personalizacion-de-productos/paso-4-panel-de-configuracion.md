# Paso 4: Panel de Configuración

### Controlador Principal

```php
/**
     * Página de configuración del módulo
     */
    public function getContent() {
        $output = '';

        // Procesar formulario
        if (Tools::isSubmit('submit' . $this->name)) {
            $labelName = (string) Tools::getValue(self::LABEL_NAME_CONFIG);
            $labelNumber = (string) Tools::getValue(self::LABEL_NUMBER_CONFIG);

            // Actualizar configuración
            Configuration::updateValue(self::LABEL_NAME_CONFIG, $labelName);
            Configuration::updateValue(self::LABEL_NUMBER_CONFIG, $labelNumber);

            // Regenerar campos con nuevas etiquetas
            $this->removeCustomizationFieldsFromAllProducts();
            $this->addCustomizationFieldsToAllProducts();

            $output = $this->displayConfirmation($this->l('Ajustes actualizados correctamente'));
        }

        return $output . $this->renderForm();
    }
```

### Formulario con HelperForm

```php
/**
     * Construye el formulario de configuración
     */
    protected function renderForm() {
        $form = [
            'form' => [
                'legend' => [
                    'title' => $this->l('Ajustes'),
                    'icon' => 'icon-cogs',
                ],
                'input' => [
                    [
                        'type' => 'text',
                        'label' => $this->l('Etiqueta para el campo \'Nombre\''),
                        'name' => self::LABEL_NAME_CONFIG,
                        'required' => true,
                        'hint' => $this->l('Texto que verá el cliente para el campo del nombre.'),
                    ],
                    [
                        'type' => 'text',
                        'label' => $this->l('Etiqueta para el campo \'Número\''),
                        'name' => self::LABEL_NUMBER_CONFIG,
                        'required' => true,
                        'hint' => $this->l('Texto que verá el cliente para el campo del número.'),
                    ],
                ],
                'submit' => [
                    'title' => $this->l('Guardar'),
                    'class' => 'btn btn-default pull-right',
                ],
            ],
        ];

        $helper = new HelperForm();

        // Configuración del helper
        $helper->table = $this->table;
        $helper->name_controller = $this->name;
        $helper->token = Tools::getAdminTokenLite('AdminModules');
        $helper->currentIndex = AdminController::$currentIndex . '&configure=' . $this->name;
        $helper->submit_action = 'submit' . $this->name;
        $helper->default_form_language = (int) Configuration::get('PS_LANG_DEFAULT');

        // Cargar valores actuales
        $helper->fields_value[self::LABEL_NAME_CONFIG] = Configuration::get(self::LABEL_NAME_CONFIG);
        $helper->fields_value[self::LABEL_NUMBER_CONFIG] = Configuration::get(self::LABEL_NUMBER_CONFIG);

        return $helper->generateForm([$form]);
    }
```
