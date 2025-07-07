# Paso 5: El Módulo Principal (quicknotes.php)

Finalmente, crearemos el archivo principal que une todo. Su trabajo es instalar y desinstalar los componentes: la tabla y el menú del Back Office.

**Acción**: Abrimos el archivo `quicknotes.php` y lo completamos.&#x20;

Métodos usados:

* `install()`: Llama a métodos auxiliares para crear la tabla (`installDb`) y el menú (`installTab`).
* `uninstall()`: Llama a los métodos para limpiarlo todo.
* `getContent()`: Redirige del botón "Configurar" a nuestro nuevo controlador.

```php
<?php
if (!defined('_PS_VERSION_')) exit;

class Quicknotes extends Module {
    const QUICKNOTES_TAB_ID = 'QUICKNOTES_TAB_ID';

    public function __construct() {
        $this->name = 'quicknotes';
        $this->tab = 'administration';
        $this->version = '1.0.0';
        $this->author = 'Desarrollo PS';
        $this->need_instance = 0;
        $this->bootstrap = true;
        parent::__construct();
        $this->displayName = $this->l('Notas Rápidas');
        $this->description = $this->l('Añade una sección de notas rápidas en el Back Office.');
    }

    public function install() {
        return parent::install() && $this->installTab() && $this->installDb();
    }

    public function uninstall() {
        return parent::uninstall() && $this->uninstallTab() && $this->uninstallDb();
    }

    public function installDb() {
        $sql = "CREATE TABLE IF NOT EXISTS `" . _DB_PREFIX_ . "quicknote` (
            `id_quicknote` INT AUTO_INCREMENT PRIMARY KEY,
            `note_text` TEXT NOT NULL,
            `active` TINYINT(1) NOT NULL DEFAULT 1,
            `date_add` DATETIME NOT NULL,
            `date_upd` DATETIME NOT NULL
        ) ENGINE=" . _MYSQL_ENGINE_ . " DEFAULT CHARSET=utf8;";
        return Db::getInstance()->execute($sql);
    }

    public function uninstallDb() {
        $sql = "DROP TABLE IF EXISTS `" . _DB_PREFIX_ . "quicknote`;";
        return Db::getInstance()->execute($sql);
    }
    
    public function installTab() {
        $tab = new Tab();
        $tab->class_name = 'AdminQuicknotes';
        $tab->module = $this->name;
        $tab->active = 1;
        $tab->id_parent = (int)Tab::getIdFromClassName('IMPROVE');
        foreach (Language::getLanguages(false) as $lang) {
            $tab->name[$lang['id_lang']] = 'Notas Rápidas';
        }
        if ($tab->add()) {
            Configuration::updateValue(self::QUICKNOTES_TAB_ID, (int)$tab->id);
            return true;
        }
        return false;
    }

    public function uninstallTab() {
        $id_tab = (int)Configuration::get(self::QUICKNOTES_TAB_ID);
        if ($id_tab) {
            $tab = new Tab($id_tab);
            if (Validate::isLoadedObject($tab)) $tab->delete();
        }
        Configuration::deleteByName(self::QUICKNOTES_TAB_ID);
        return true;
    }

    public function getContent() {
        Tools::redirectAdmin($this->context->link->getAdminLink('AdminQuicknotes'));
    }
}
```
