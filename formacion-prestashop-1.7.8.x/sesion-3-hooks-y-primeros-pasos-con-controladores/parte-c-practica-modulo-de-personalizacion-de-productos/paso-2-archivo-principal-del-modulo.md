# Paso 2: Archivo Principal del Módulo

Crea el archivo `productpersonalization.php` en la raíz del módulo:

### Constructor de la Clase

```php
<?php
if (!defined('_PS_VERSION_')) {
    exit;
}

class ProductPersonalization extends Module {
    // Constantes para configuración (buena práctica)
    const LABEL_NAME_CONFIG = 'PRODUCTPERSONALIZATION_LABEL_NAME';
    const LABEL_NUMBER_CONFIG = 'PRODUCTPERSONALIZATION_LABEL_NUMBER';

    public function __construct() {
        $this->name = 'productpersonalization';
        $this->version = '1.0.0';
        $this->author = 'Desarrollo PS';
        $this->need_instance = 0;
        $this->ps_versions_compliancy = [
            'min' => '1.7.8.0',
            'max' => _PS_VERSION_,
        ];
        $this->bootstrap = true; // ¡Importante para el panel de configuración!

        parent::__construct();

        // Textos traducibles
        $this->displayName = $this->l('Personalización de Producto');
        $this->description = $this->l('Añade campos para nombre y número a los productos.');
        $this->confirmUninstall = $this->l('¿Estás seguro de que quieres desinstalar? Se eliminarán todos los campos de personalización.');
    }
```

### Métodos de Instalación y Desinstalación

```php
/**
     * Proceso de instalación del módulo
     * @return bool
     */
    public function install() {
        if (!parent::install()) {
            return false;
        }

        // Configurar valores por defecto
        Configuration::updateValue(self::LABEL_NAME_CONFIG, $this->l('Introduce tu nombre'));
        Configuration::updateValue(self::LABEL_NUMBER_CONFIG, $this->l('Introduce tu número'));

        // Añadir campos a todos los productos existentes
        $this->addCustomizationFieldsToAllProducts();

        return true;
    }

    /**
     * Proceso de desinstalación del módulo
     * @return bool
     */
    public function uninstall() {
        if (!parent::uninstall()) {
            return false;
        }

        // Eliminar configuración
        Configuration::deleteByName(self::LABEL_NAME_CONFIG);
        Configuration::deleteByName(self::LABEL_NUMBER_CONFIG);

        // Eliminar todos los campos de personalización
        $this->removeCustomizationFieldsFromAllProducts();

        return true;
    }

```
