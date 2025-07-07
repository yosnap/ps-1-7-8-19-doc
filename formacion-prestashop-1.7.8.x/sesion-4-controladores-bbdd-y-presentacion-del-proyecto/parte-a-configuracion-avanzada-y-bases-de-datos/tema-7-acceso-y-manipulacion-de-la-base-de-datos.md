# Tema #7 - Acceso y Manipulación de la Base de Datos

### **1. Instalación de Tablas con `install()`**

Un módulo debe crear sus propias estructuras de datos al instalarse y limpiarlas al desinstalarse.

* Explicación: Se crean métodos auxiliares dentro de la clase principal del módulo que son llamados desde `install()` y `uninstall()`.
* Ejercicio: Añade a tu módulo un método `install()` que cree una tabla `ps_configavanzada_log` para guardar un historial de quién y cuándo ha guardado la configuración.

```php
// En la clase principal del módulo: configuracionavanzada.php
public function install()
{
    return parent::install() && $this->createTables();
}

public function uninstall()
{
    return parent::uninstall() && $this->dropTables();
}

private function createTables() {
    $sql = "CREATE TABLE IF NOT EXISTS `" . _DB_PREFIX_ . "configavanzada_log` (
    `id_log` INT AUTO_INCREMENT PRIMARY KEY,
    `id_employee` INT NOT NULL,
    `change_details` VARCHAR(255),
    `date_add` DATETIME NOT NULL
) ENGINE=" . _MYSQL_ENGINE_ . " DEFAULT CHARSET=utf8;";
    return Db::getInstance()->execute($sql);
}

private function dropTables() {
    $sql = "DROP TABLE IF EXISTS `" . _DB_PREFIX_ . "configavanzada_log`;";
    return Db::getInstance()->execute($sql);
}
```

### **2. `Db::getInstance()` vs. `ObjectModel` (ORM)**

Son las dos formas principales de interactuar con la base de datos.

* Explicación:
  * `Db::getInstance()`: Para ejecutar consultas SQL directas. Es rápido y flexible.
    * `execute()`: Para `INSERT`, `UPDATE`, `DELETE`. Devuelve `true`/`false`.
    * `executeS()`: Para `SELECT` de múltiples filas. Devuelve un `array` de resultados.
    * `getValue()`: Para `SELECT` que devuelve un único valor.
  * `ObjectModel` (ORM): Para tratar una fila de una tabla como si fuera un objeto PHP. Es más limpio, seguro y orientado a objetos, ideal para CRUD (Crear, Leer, Actualizar, Borrar).
* Ejercicio:
  1. Crea un `ObjectModel`: Crea una nueva clase `ConfigAvanzadaLog` en `modules/configuracionavanzada/classes/ConfigAvanzadaLog.php` que extienda `ObjectModel` y defina su estructura.
  2. Úsalo: Modifica el método `postProcess()` del controlador. Después de guardar la configuración, instancia tu nuevo `ObjectModel` para guardar un registro en tu tabla personalizada.

Ejemplo usando ambos casos:

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

// ObjectModel integrado en el mismo archivo
class ConfigAvanzadaLog extends ObjectModel {
    public $id_config_log;
    public $config_key;
    public $old_value;
    public $new_value;
    public $id_employee;
    public $date_add;
    public $date_upd;

    public static $definition = [
        'table' => 'config_avanzada_log',
        'primary' => 'id_config_log',
        'fields' => [
            'config_key' => ['type' => self::TYPE_STRING, 'validate' => 'isCleanHtml', 'required' => true, 'size' => 100],
            'old_value' => ['type' => self::TYPE_STRING, 'validate' => 'isCleanHtml', 'size' => 255],
            'new_value' => ['type' => self::TYPE_STRING, 'validate' => 'isCleanHtml', 'required' => true, 'size' => 255],
            'id_employee' => ['type' => self::TYPE_INT, 'validate' => 'isUnsignedId'],
            'date_add' => ['type' => self::TYPE_DATE, 'validate' => 'isDate'],
            'date_upd' => ['type' => self::TYPE_DATE, 'validate' => 'isDate'],
        ],
    ];

    public static function createLog($configKey, $oldValue, $newValue, $idEmployee = null) {
        $log = new self();
        $log->config_key = $configKey;
        $log->old_value = $oldValue;
        $log->new_value = $newValue;
        $log->id_employee = $idEmployee ?: (isset(Context::getContext()->employee) ? Context::getContext()->employee->id : 0);
        return $log->add();
    }

    public static function getRecentLogs($limit = 10) {
        $sql = new DbQuery();
        $sql->select('cal.*, CONCAT(e.firstname, " ", e.lastname) as employee_name');
        $sql->from('config_avanzada_log', 'cal');
        $sql->leftJoin('employee', 'e', 'e.id_employee = cal.id_employee');
        $sql->orderBy('cal.date_add DESC');
        $sql->limit($limit);
        return Db::getInstance()->executeS($sql);
    }
}

// Clase del formulario
class SimpleFormType extends AbstractType {
    public function buildForm(FormBuilderInterface $builder, array $options) {
        $builder
            ->add('titulo', TextType::class, [
                'label' => 'Título del Banner',
                'required' => true,
                'data' => $options['data']['titulo'] ?? '',
            ])
            ->add('save', SubmitType::class, [
                'label' => 'Guardar',
                'attr' => ['class' => 'btn btn-primary']
            ]);
    }
}

class ConfiguracionAvanzada extends Module {

    const CONFIG_AVANZADA_TITULO = 'CONFIG_AVANZADA_TITULO';

    public function __construct() {
        $this->name = 'configuracionavanzada';
        $this->tab = 'others';
        $this->version = '1.0.0';
        $this->author = 'Desarrollo PS';
        $this->need_instance = 0;
        $this->bootstrap = true;

        parent::__construct();

        $this->displayName = $this->l('Configuración Avanzada (Symfony)');
        $this->description = $this->l('Módulo con FormBuilder de Symfony y ObjectModel.');
        $this->ps_versions_compliancy = ['min' => '1.7.7', 'max' => _PS_VERSION_];
    }

    public function install() {
        return parent::install() &&
            $this->installDb() &&
            Configuration::updateValue(self::CONFIG_AVANZADA_TITULO, $this->l('Título por defecto'));
    }

    public function uninstall() {
        return parent::uninstall() &&
            $this->uninstallDb() &&
            Configuration::deleteByName(self::CONFIG_AVANZADA_TITULO);
    }

    private function installDb() {
        $sql = 'CREATE TABLE IF NOT EXISTS `' . _DB_PREFIX_ . 'config_avanzada_log` (
            `id_config_log` int(11) NOT NULL AUTO_INCREMENT,
            `config_key` varchar(100) NOT NULL,
            `old_value` varchar(255) DEFAULT NULL,
            `new_value` varchar(255) NOT NULL,
            `id_employee` int(11) DEFAULT NULL,
            `date_add` datetime NOT NULL,
            `date_upd` datetime NOT NULL,
            PRIMARY KEY (`id_config_log`),
            KEY `config_key` (`config_key`),
            KEY `id_employee` (`id_employee`)
        ) ENGINE=' . _MYSQL_ENGINE_ . ' DEFAULT CHARSET=utf8;';

        return Db::getInstance()->execute($sql);
    }

    private function uninstallDb() {
        $sql = 'DROP TABLE IF EXISTS `' . _DB_PREFIX_ . 'config_avanzada_log`';
        return Db::getInstance()->execute($sql);
    }

    // ... resto del código igual que antes (getContent, renderSymfonyForm, etc.)
    public function getContent() {
        $valorActual = Configuration::get(self::CONFIG_AVANZADA_TITULO, '');

        try {
            $request = Request::createFromGlobals();
            $form = $this->get('form.factory')->create(SimpleFormType::class, [
                'titulo' => $valorActual
            ]);

            $form->handleRequest($request);
            $output = '';

            if ($form->isSubmitted() && $form->isValid()) {
                $data = $form->getData();
                $nuevoTitulo = $data['titulo'];

                ConfigAvanzadaLog::createLog(
                    self::CONFIG_AVANZADA_TITULO,
                    $valorActual,
                    $nuevoTitulo,
                    $this->context->employee->id
                );

                Configuration::updateValue(self::CONFIG_AVANZADA_TITULO, $nuevoTitulo);
                $output = $this->displayConfirmation($this->l('Configuración guardada y registrada en el log'));
                $valorActual = $nuevoTitulo;
            } elseif ($form->isSubmitted()) {
                $output = $this->displayError($this->l('Error en el formulario'));
            }

            return $output . $this->renderSymfonyForm($form) . $this->renderLogsTable();
        } catch (Exception $e) {
            return $this->renderFallbackForm();
        }
    }

    /**
     * Renderizar formulario con Symfony
     */
    private function renderSymfonyForm($form) {
        $formView = $form->createView();
        $valorActual = Configuration::get(self::CONFIG_AVANZADA_TITULO, '');

        $html = '
        <div class="panel">
            <div class="panel-heading">
                <i class="icon-cogs"></i> ' . $this->l('Configuración con Symfony FormBuilder') . '
            </div>
            <div class="panel-body">
                <div class="alert alert-info">
                    <strong>Valor actual:</strong> ' . htmlspecialchars($valorActual) . '
                </div>';

        $html .= '<form method="post" action="' . htmlspecialchars($_SERVER['REQUEST_URI']) . '">';

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
     * Renderizar tabla de logs
     */
    private function renderLogsTable() {
        $logs = ConfigAvanzadaLog::getRecentLogs(5);

        $html = '
        <div class="panel">
            <div class="panel-heading">
                <i class="icon-list"></i> ' . $this->l('Historial de Cambios (ObjectModel)') . '
            </div>
            <div class="panel-body">';

        if (empty($logs)) {
            $html .= '<div class="alert alert-info">' . $this->l('No hay cambios registrados') . '</div>';
        } else {
            $html .= '
                <table class="table table-striped">
                    <thead>
                        <tr>
                            <th>' . $this->l('Configuración') . '</th>
                            <th>' . $this->l('Valor Anterior') . '</th>
                            <th>' . $this->l('Valor Nuevo') . '</th>
                            <th>' . $this->l('Usuario') . '</th>
                            <th>' . $this->l('Fecha') . '</th>
                        </tr>
                    </thead>
                    <tbody>';

            foreach ($logs as $log) {
                $html .= '
                        <tr>
                            <td>' . htmlspecialchars($log['config_key']) . '</td>
                            <td>' . htmlspecialchars($log['old_value'] ?: '-') . '</td>
                            <td>' . htmlspecialchars($log['new_value']) . '</td>
                            <td>' . htmlspecialchars($log['employee_name'] ?: 'Sistema') . '</td>
                            <td>' . htmlspecialchars($log['date_add']) . '</td>
                        </tr>';
            }

            $html .= '
                    </tbody>
                </table>';
        }

        $html .= '
            </div>
        </div>';

        return $html;
    }

    /**
     * Formulario de respaldo si Symfony falla
     */
    private function renderFallbackForm() {
        $valorActual = Configuration::get(self::CONFIG_AVANZADA_TITULO, '');

        if (Tools::isSubmit('submitFallback')) {
            $titulo = Tools::getValue('CONFIG_AVANZADA_TITULO');
            if (!empty($titulo)) {
                // **USAR OBJECTMODEL**: También en el fallback
                ConfigAvanzadaLog::createLog(
                    self::CONFIG_AVANZADA_TITULO,
                    $valorActual,
                    $titulo,
                    $this->context->employee->id
                );

                Configuration::updateValue(self::CONFIG_AVANZADA_TITULO, $titulo);
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

        return $output . $html . $this->renderLogsTable();
    }
}

```
