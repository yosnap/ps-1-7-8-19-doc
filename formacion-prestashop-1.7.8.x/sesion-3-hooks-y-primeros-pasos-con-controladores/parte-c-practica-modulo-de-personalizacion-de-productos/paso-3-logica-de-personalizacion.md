# Paso 3: Lógica de Personalización

### Añadir Campos a Productos

```php
/**
     * Añade campos de personalización a todos los productos activos
     * @return bool
     */
    private function addCustomizationFieldsToAllProducts() {
        $products = Product::getProducts($this->context->language->id, 0, 0, 'id_product', 'ASC', false, true);

        $labelName = Configuration::get(self::LABEL_NAME_CONFIG, $this->context->language->id);
        $labelNumber = Configuration::get(self::LABEL_NUMBER_CONFIG, $this->context->language->id);

        foreach ($products as $productRow) {
            $product = new Product($productRow['id_product']);

            // Habilitar personalización si no está activa
            if (!$product->customizable) {
                $product->customizable = 1;
                $product->save();
            }

            // Crear campos
            $this->createCustomizationField($product->id, $labelName, 20);
            $this->createCustomizationField($product->id, $labelNumber, 2);
        }

        return true;
    }

    /**
     * Crea un campo de personalización específico
     * @param int $id_product ID del producto
     * @param string $label Etiqueta del campo
     * @param int $maxLength Longitud máxima (informativo)
     */
    private function createCustomizationField($id_product, $label, $maxLength) {
        $field = new CustomizationField();
        $field->id_product = (int)$id_product;
        $field->type = (int)Product::CUSTOMIZE_TEXTFIELD;
        $field->required = 0;

        // Etiqueta multi-idioma
        foreach (Language::getLanguages(true) as $lang) {
            $field->name[$lang['id_lang']] = $label;
        }

        $field->add();
    }
```

### Eliminar Campos

```php
/**
     * Elimina todos los campos de personalización de todos los productos
     * @return bool
     */
    private function removeCustomizationFieldsFromAllProducts() {
        $products = Product::getProducts($this->context->language->id, 0, 0, 'id_product', 'ASC', false, true);

        foreach ($products as $productRow) {
            $this->removeCustomizationFieldsFromProduct($productRow['id_product']);
        }

        return true;
    }

    /**
     * Elimina los campos de personalización de un producto específico
     * @param int $id_product ID del producto
     */
    private function removeCustomizationFieldsFromProduct($id_product) {
        $db = Db::getInstance();

        // Primero obtener los IDs de los campos de personalización para este producto
        $customizationFields = $db->executeS(
            '
            SELECT id_customization_field
            FROM ' . _DB_PREFIX_ . 'customization_field
            WHERE id_product = ' . (int)$id_product
        );

        if ($customizationFields) {
            // Eliminar las traducciones de los campos
            foreach ($customizationFields as $field) {
                $db->delete('customization_field_lang', 'id_customization_field = ' . (int)$field['id_customization_field']);
            }
        }

        // Eliminar los campos de personalización del producto
        $db->delete('customization_field', 'id_product = ' . (int)$id_product);

        // Deshabilitar personalización si no hay más campos
        $product = new Product($id_product);
        $product->customizable = 0;
        $product->save();
    }
```
