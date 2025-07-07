# Paso 3: Archivo Principal del Módulo

Crea el archivo `welcomelogin.php` con la clase principal:

### Constructor y Propiedades

```php
<?php

if (!defined('_PS_VERSION_')) {
    exit;
}

class WelcomeLogin extends Module {
    public function __construct() {
        $this->name = 'welcomelogin';
        $this->tab = 'front_office_features';
        $this->version = '1.0.0';
        $this->author = 'Desarrollo PS';
        $this->need_instance = 1;
        $this->bootstrap = true;

        parent::__construct();

        $this->displayName = $this->l('Módulo Saludo de Bienvenida');
        $this->description = $this->l('Muestra un saludo personalizado a los clientes que inician sesión.');
        $this->ps_versions_compliancy = ['min' => '1.7', 'max' => _PS_VERSION_];
    }
}
```

### Métodos de Instalación y Desinstalación

```php
/**
     * Instalación del módulo
     */
    public function install() {
        return parent::install() &&
            $this->registerHook('actionAuthentication') && // Detectar login
            $this->registerHook('displayHome') &&          // Mostrar mensaje
            $this->registerHook('displayHeader') &&        // Cargar CSS
            $this->createDbTables();                       // Crear tabla
    }

    /**
     * Desinstalación del módulo
     */
    public function uninstall() {
        return parent::uninstall() && $this->removeDbTables();
    }
```

### Gestión de Base de Datos

```php
/**
     * Crea la tabla personalizada para el log de logins
     */
    public function createDbTables() {
        $sql = "CREATE TABLE IF NOT EXISTS `" . _DB_PREFIX_ . "customer_login_log` (
            `id_customer` INT(10) UNSIGNED NOT NULL,
            `last_login` DATETIME NOT NULL,
            PRIMARY KEY (`id_customer`)
        ) ENGINE=" . _MYSQL_ENGINE_ . " DEFAULT CHARSET=utf8;";

        return Db::getInstance()->execute($sql);
    }

    /**
     * Método auxiliar para forzar la creación de la tabla (para debugging)
     */
    public function forceCreateTable() {
        try {
            $sql = "CREATE TABLE IF NOT EXISTS `" . _DB_PREFIX_ . "customer_login_log` (
                `id_customer` INT(10) UNSIGNED NOT NULL,
                `last_login` DATETIME NOT NULL,
                PRIMARY KEY (`id_customer`)
            ) ENGINE=" . _MYSQL_ENGINE_ . " DEFAULT CHARSET=utf8;";

            $result = Db::getInstance()->execute($sql);

            if ($result) {
                return "Tabla creada exitosamente";
            } else {
                return "Error al crear la tabla: " . Db::getInstance()->getMsgError();
            }
        } catch (Exception $e) {
            return "Excepción: " . $e->getMessage();
        }
    }

    /**
     * Método para verificar si la tabla existe
     */
    public function checkTableExists() {
        $sql = "SHOW TABLES LIKE '" . _DB_PREFIX_ . "customer_login_log'";
        $result = Db::getInstance()->executeS($sql);
        return !empty($result);
    }

    /**
     * Elimina la tabla personalizada
     */
    public function removeDbTables() {
        $sql = "DROP TABLE IF EXISTS `" . _DB_PREFIX_ . "customer_login_log`;";
        return Db::getInstance()->execute($sql);
    }
```
