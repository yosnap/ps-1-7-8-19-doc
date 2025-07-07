# PARTE 1: Desarrollo de Módulo de Testimonios PrestaShop 1.7.8.x

***

### 📋 Objetivos de la Clase

Al finalizar esta clase, los participantes serán capaces de:

1. **Entender la estructura** básica de un módulo PrestaShop 1.7.8.x
2. **Crear desde cero** la estructura inicial del módulo
3. **Implementar la clase principal** del módulo con métodos básicos
4. **Crear la tabla de base de datos** necesaria para almacenar testimonios
5. **Implementar hooks básicos** para integración con PrestaShop
6. **Desarrollar el panel de administración** con configuración inicial

***

### 🏗️ Estructura del Módulo a Desarrollar

#### Características del Módulo "GestorTestimonios"

* **Nombre**: gestortestimonios
* **Versión**: 1.0.0
* **Compatibilidad**: PrestaShop 1.7.0.0 - 1.7.8.x
* **Funcionalidad**: Gestión completa de testimonios de clientes

#### Funcionalidades Principales

1. **Sistema de testimonios** con campos: nombre, empresa, email, texto, rating
2. **Panel de administración** para aprobar/rechazar testimonios
3. **Vista pública** para mostrar testimonios aprobados
4. **Formulario frontend** para envío de nuevos testimonios
5. **Sistema de configuración** flexible
6. **Integración con hooks** de PrestaShop

***

### 📁 PASO 1: Estructura de Directorios (15 min)

#### 1.1 Crear la estructura básica

```
gestortestimonios/
├── gestortestimonios.php          # Clase principal del módulo
├── index.php                      # Archivo de seguridad
├── config_es.xml                  # Configuración de idioma
├── controllers/
│   ├── index.php                  # Archivo de seguridad
│   ├── admin/
│   │   └── index.php              # Archivo de seguridad
│   └── front/
│       ├── index.php              # Archivo de seguridad
│       ├── testimonios.php        # Controlador listado
│       └── enviar.php             # Controlador formulario
├── translations/
│   ├── index.php                  # Archivo de seguridad
│   └── es.php                     # Traducciones español
├── views/
│   ├── index.php                  # Archivo de seguridad
│   ├── css/
│   │   ├── index.php              # Archivo de seguridad
│   │   └── gestortestimonios.css  # Estilos del módulo
│   ├── js/
│   │   ├── index.php              # Archivo de seguridad
│   │   └── gestortestimonios.js   # JavaScript del módulo
│   └── templates/
│       ├── index.php              # Archivo de seguridad
│       ├── admin/
│       │   ├── index.php          # Archivo de seguridad
│       │   └── configure.tpl      # Template configuración
│       ├── front/
│       │   ├── testimonios.tpl    # Template listado
│       │   └── enviar.tpl         # Template formulario
│       └── hook/
│           ├── index.php          # Archivo de seguridad
│           ├── testimonios-home.tpl    # Hook homepage
│           └── testimonios-footer.tpl  # Hook footer
└── tests/
    ├── index.php                  # Archivo de seguridad
    └── TestimonioValidationTest.php # Tests unitarios
```

#### 1.2 Crear archivos de seguridad index.php

**Contenido para todos los archivos index.php:**

```php
<?php
/**
 * NOTICE OF LICENSE
 *
 * This file is licenced under the Software License Agreement.
 * With the purchase or the installation of the software in your application
 * you accept the licence agreement.
 */

header('Expires: Mon, 26 Jul 1997 05:00:00 GMT');
header('Last-Modified: ' . gmdate('D, d M Y H:i:s') . ' GMT');

header('Cache-Control: no-store, no-cache, must-revalidate');
header('Cache-Control: post-check=0, pre-check=0', false);
header('Pragma: no-cache');

header('Location: ../');
exit;
```

***

### 💻 PASO 2: Clase Principal del Módulo (30 min)

#### 2.1 Crear gestortestimonios.php

```php
<?php

/**
 * Módulo Gestor de Testimonios de Clientes para PrestaShop 1.7.8.x
 * 
 * @author Desarrollo PS
 * @version 1.0.0
 * @license AFL 3.0
 * @copyright 2024
 */

if (!defined('_PS_VERSION_')) {
    exit;
}

/**
 * Clase principal del módulo Gestor de Testimonios
 *
 * @extends Module
 */
class GestorTestimonios extends Module {
    
    public function __construct() {
        $this->name = 'gestortestimonios';
        $this->tab = 'front_office_features';
        $this->version = '1.0.0';
        $this->author = 'Desarrollo PS';
        $this->need_instance = 0;
        $this->ps_versions_compliancy = [
            'min' => '1.7.0.0',
            'max' => _PS_VERSION_
        ];
        $this->bootstrap = true;

        parent::__construct();

        $this->displayName = $this->l('Gestor de Testimonios de Clientes');
        $this->description = $this->l('Módulo profesional para gestionar testimonios de clientes con sistema de aprobación, rating y panel de administración completo.');

        $this->confirmUninstall = $this->l('¿Estás seguro de que quieres desinstalar este módulo? Se perderán todos los testimonios almacenados.');
    }

    /**
     * Instalación del módulo
     */
    public function install() {
        $install_result = parent::install();

        if (!$install_result) {
            return false;
        }

        // Registrar hooks necesarios
        $hooks = [
            'displayHeader',           // Para cargar CSS/JS
            'displayHome',            // Mostrar testimonios en home
            'displayFooter',          // Mostrar testimonios en footer
            'displayCustomerAccount', // Enlace en cuenta cliente
        ];

        foreach ($hooks as $hook) {
            if (!$this->registerHook($hook)) {
                return false;
            }
        }

        // Crear tabla de testimonios
        if (!$this->createTestimoniosTable()) {
            return false;
        }

        // Configuración inicial
        Configuration::updateValue('GESTOR_TESTIMONIOS_ENABLED', 1);
        Configuration::updateValue('GESTOR_TESTIMONIOS_AUTO_APPROVE', 0);
        Configuration::updateValue('GESTOR_TESTIMONIOS_MAX_PER_PAGE', 5);
        Configuration::updateValue('GESTOR_TESTIMONIOS_MIN_RATING', 1);
        Configuration::updateValue('GESTOR_TESTIMONIOS_MAX_RATING', 5);

        return true;
    }

    /**
     * Desinstalación del módulo
     */
    public function uninstall() {
        // Eliminar tabla de testimonios
        $this->dropTestimoniosTable();

        // Eliminar configuraciones
        Configuration::deleteByName('GESTOR_TESTIMONIOS_ENABLED');
        Configuration::deleteByName('GESTOR_TESTIMONIOS_AUTO_APPROVE');
        Configuration::deleteByName('GESTOR_TESTIMONIOS_MAX_PER_PAGE');
        Configuration::deleteByName('GESTOR_TESTIMONIOS_MIN_RATING');
        Configuration::deleteByName('GESTOR_TESTIMONIOS_MAX_RATING');

        return parent::uninstall();
    }
}
```

#### 2.2 Explicación de los componentes

{% hint style="info" %}
**Constructor:**

* Definimos metadatos básicos del módulo
* Configuramos compatibilidad con PrestaShop 1.7.x
* Establecemos mensajes de confirmación&#x20;
{% endhint %}

{% hint style="warning" %}
**Método install():**

* Registra hooks necesarios para la integración
* Crea la tabla de base de datos
* Establece configuración inicial&#x20;
{% endhint %}

{% hint style="danger" %}
**Método uninstall():**

* Limpia tabla de base de datos
* Elimina configuraciones del módulo
{% endhint %}

***

### 🗄️ PASO 3: Creación de la Base de Datos

#### 3.1 Añadir métodos de base de datos a gestortestimonios.php

```php
/**
 * Crear tabla de testimonios según especificaciones
 */
private function createTestimoniosTable() {
    $sql = 'CREATE TABLE IF NOT EXISTS `' . _DB_PREFIX_ . 'testimonios` (
        `id` int(11) NOT NULL AUTO_INCREMENT,
        `client_name` varchar(255) NOT NULL,
        `company` varchar(255) NULL,
        `testimonial_text` text NOT NULL,
        `rating` tinyint(1) UNSIGNED NULL,
        `status` enum("pending", "approved", "rejected") NOT NULL DEFAULT "pending",
        `created_at` timestamp DEFAULT CURRENT_TIMESTAMP,
        `customer_id` int(11) NULL,
        `email` varchar(255) NULL,
        `display_order` int(11) DEFAULT 0,
        PRIMARY KEY (`id`),
        KEY `idx_status` (`status`),
        KEY `idx_created` (`created_at`),
        KEY `idx_rating` (`rating`),
        CONSTRAINT `chk_rating` CHECK (`rating` IS NULL OR (`rating` >= 1 AND `rating` <= 5))
    ) ENGINE=' . _MYSQL_ENGINE_ . ' DEFAULT CHARSET=utf8mb4;';

    $result = Db::getInstance()->execute($sql);

    if (!$result) {
        $error = Db::getInstance()->getMsgError();
        PrestaShopLogger::addLog(
            'GestorTestimonios: Error creating table - ' . $error,
            3,
            null,
            'GestorTestimonios'
        );
    }

    return $result;
}

/**
 * Eliminar tabla de testimonios
 */
private function dropTestimoniosTable() {
    $sql = 'DROP TABLE IF EXISTS `' . _DB_PREFIX_ . 'testimonios`';
    return Db::getInstance()->execute($sql);
}
```

#### 3.2 Estructura de la tabla explicada

{% tabs %}
{% tab title="Campos principales" %}
* `id`: Clave primaria autoincremental
* `client_name`: Nombre del cliente (obligatorio)
* `company`: Empresa u organización (opcional)
* `testimonial_text`: Texto del testimonio (obligatorio)
* `rating`: Valoración 1-5 estrellas (opcional)
* `status`: Estado del testimonio (pending/approved/rejected)
* `created_at`: Fecha de creación automática
* `customer_id`: Relación con clientes registrados
* `email`: Email del cliente
* `display_order`: Orden de visualización
{% endtab %}

{% tab title="Índices para optimización" %}
* Índice en `status` para consultas de testimonios aprobados
* Índice en `created_at` para ordenación cronológica
* Índice en `rating` para estadísticas
{% endtab %}

{% tab title="Restricciones" %}
Constraint para validar rango de rating (1-5)&#x20;
{% endtab %}
{% endtabs %}

***

### 🎣 PASO 4: Implementación de Hooks Básicos

#### 4.1 Hook displayHeader

```php
/**
 * Hook: displayHeader
 * Cargar CSS y JS necesarios
 */
public function hookDisplayHeader() {
    // Solo cargar en páginas relevantes
    $controller = $this->context->controller;
    $page_name = get_class($controller);

    if (
        $page_name === 'IndexController' ||
        $page_name === 'TestimoniosController' ||
        $this->isTestimoniosPage()
    ) {
        $this->context->controller->addCSS($this->_path . 'views/css/gestortestimonios.css');
        $this->context->controller->addJS($this->_path . 'views/js/gestortestimonios.js');
    }
}
```

#### 4.2 Hook displayHome

```php
/**
 * Hook: displayHome
 * Mostrar testimonios aprobados en la página de inicio
 */
public function hookDisplayHome() {
    if (!Configuration::get('GESTOR_TESTIMONIOS_ENABLED')) {
        return '';
    }

    $testimonios = $this->getApprovedTestimonios(
        Configuration::get('GESTOR_TESTIMONIOS_MAX_PER_PAGE')
    );

    if (empty($testimonios)) {
        return '';
    }

    $this->context->smarty->assign([
        'testimonios' => $testimonios,
        'show_rating' => true,
        'module_dir' => $this->_path
    ]);

    return $this->display(__FILE__, 'views/templates/hook/testimonios-home.tpl');
}
```

#### 4.3 Hook displayFooter

```php
/**
 * Hook: displayFooter
 * Mostrar enlace para enviar testimonio
 */
public function hookDisplayFooter() {
    if (!Configuration::get('GESTOR_TESTIMONIOS_ENABLED')) {
        return '';
    }

    $this->context->smarty->assign([
        'testimonios_link' => $this->context->link->getModuleLink(
            'gestortestimonios',
            'testimonios'
        ),
        'enviar_testimonio_link' => $this->context->link->getModuleLink(
            'gestortestimonios',
            'enviar'
        )
    ]);

    return $this->display(__FILE__, 'views/templates/hook/testimonios-footer.tpl');
}
```

#### 4.4 Métodos auxiliares para hooks

```php
/**
 * Obtener testimonios aprobados
 */
public function getApprovedTestimonios($limit = 10, $random = false) {
    $orderBy = $random ? 'RAND()' : 't.created_at DESC';

    $sql = 'SELECT t.*, c.firstname, c.lastname
            FROM `' . _DB_PREFIX_ . 'testimonios` t
            LEFT JOIN `' . _DB_PREFIX_ . 'customer` c ON t.customer_id = c.id_customer
            WHERE t.status = "approved"
            ORDER BY ' . $orderBy . '
            LIMIT ' . (int)$limit;

    return Db::getInstance()->executeS($sql);
}

/**
 * Verificar si estamos en página de testimonios
 */
private function isTestimoniosPage() {
    return Tools::getValue('controller') === 'testimonios' ||
           Tools::getValue('controller') === 'enviar';
}
```

***

### ⚙️ PASO 5: Panel de Administración Básico

#### 5.1 Método getContent() para configuración

```php
/**
 * Página de configuración del módulo
 */
public function getContent() {
    $output = '';

    // Procesar formulario de configuración
    if (Tools::isSubmit('submit' . $this->name)) {
        $output .= $this->processConfiguration();
    }

    return $output . $this->displayConfigurationForm();
}

/**
 * Procesar configuración del módulo
 */
private function processConfiguration() {
    $enabled = Tools::getValue('GESTOR_TESTIMONIOS_ENABLED');
    $auto_approve = Tools::getValue('GESTOR_TESTIMONIOS_AUTO_APPROVE');
    $max_per_page = (int)Tools::getValue('GESTOR_TESTIMONIOS_MAX_PER_PAGE');
    $min_rating = (int)Tools::getValue('GESTOR_TESTIMONIOS_MIN_RATING');
    $max_rating = (int)Tools::getValue('GESTOR_TESTIMONIOS_MAX_RATING');

    // Validaciones
    if ($max_per_page < 1 || $max_per_page > 50) {
        return $this->displayError($this->l('El número de testimonios por página debe estar entre 1 y 50.'));
    }

    if ($min_rating < 1 || $min_rating > 5 || $max_rating < 1 || $max_rating > 5 || $min_rating >= $max_rating) {
        return $this->displayError($this->l('Las valoraciones deben estar entre 1 y 5, y el mínimo debe ser menor que el máximo.'));
    }

    // Guardar configuración
    Configuration::updateValue('GESTOR_TESTIMONIOS_ENABLED', $enabled);
    Configuration::updateValue('GESTOR_TESTIMONIOS_AUTO_APPROVE', $auto_approve);
    Configuration::updateValue('GESTOR_TESTIMONIOS_MAX_PER_PAGE', $max_per_page);
    Configuration::updateValue('GESTOR_TESTIMONIOS_MIN_RATING', $min_rating);
    Configuration::updateValue('GESTOR_TESTIMONIOS_MAX_RATING', $max_rating);

    return $this->displayConfirmation($this->l('Configuración guardada exitosamente.'));
}
```

#### 5.2 Crear template de configuración básico

**Archivo: views/templates/admin/configure.tpl**

```smarty
<div class="panel">
    <div class="panel-heading">
        <i class="icon-cogs"></i>
        {l s='Configuración del Módulo' mod='gestortestimonios'}
    </div>
    
    <form id="configuration_form" class="defaultForm form-horizontal" 
          action="{$smarty.server.REQUEST_URI}" method="post" enctype="multipart/form-data">
        
        <div class="form-group">
            <label class="control-label col-lg-3">
                {l s='Habilitar módulo' mod='gestortestimonios'}
            </label>
            <div class="col-lg-9">
                <span class="switch prestashop-switch fixed-width-lg">
                    <input type="radio" name="GESTOR_TESTIMONIOS_ENABLED" id="enabled_on" value="1" 
                           {if $gestor_enabled}checked="checked"{/if} />
                    <label for="enabled_on">{l s='Sí' mod='gestortestimonios'}</label>
                    <input type="radio" name="GESTOR_TESTIMONIOS_ENABLED" id="enabled_off" value="0" 
                           {if !$gestor_enabled}checked="checked"{/if} />
                    <label for="enabled_off">{l s='No' mod='gestortestimonios'}</label>
                    <a class="slide-button btn"></a>
                </span>
            </div>
        </div>

        <div class="form-group">
            <label class="control-label col-lg-3">
                {l s='Auto-aprobar testimonios' mod='gestortestimonios'}
            </label>
            <div class="col-lg-9">
                <span class="switch prestashop-switch fixed-width-lg">
                    <input type="radio" name="GESTOR_TESTIMONIOS_AUTO_APPROVE" id="auto_approve_on" value="1" 
                           {if $auto_approve}checked="checked"{/if} />
                    <label for="auto_approve_on">{l s='Sí' mod='gestortestimonios'}</label>
                    <input type="radio" name="GESTOR_TESTIMONIOS_AUTO_APPROVE" id="auto_approve_off" value="0" 
                           {if !$auto_approve}checked="checked"{/if} />
                    <label for="auto_approve_off">{l s='No' mod='gestortestimonios'}</label>
                    <a class="slide-button btn"></a>
                </span>
            </div>
        </div>

        <div class="form-group">
            <label class="control-label col-lg-3">
                {l s='Testimonios por página' mod='gestortestimonios'}
            </label>
            <div class="col-lg-9">
                <input type="number" name="GESTOR_TESTIMONIOS_MAX_PER_PAGE" 
                       value="{$max_per_page}" min="1" max="50" />
            </div>
        </div>

        <div class="panel-footer">
            <button type="submit" value="1" id="configuration_form_submit_btn" 
                    name="{$submit_action}" class="btn btn-default pull-right">
                <i class="process-icon-save"></i> {l s='Guardar' mod='gestortestimonios'}
            </button>
        </div>
    </form>
</div>
```

#### 5.3 Método para mostrar formulario

```php
/**
 * Mostrar formulario de configuración
 */
private function displayConfigurationForm() {
    $this->context->smarty->assign([
        'module_dir' => $this->_path,
        'gestor_enabled' => Configuration::get('GESTOR_TESTIMONIOS_ENABLED'),
        'auto_approve' => Configuration::get('GESTOR_TESTIMONIOS_AUTO_APPROVE'),
        'max_per_page' => Configuration::get('GESTOR_TESTIMONIOS_MAX_PER_PAGE'),
        'min_rating' => Configuration::get('GESTOR_TESTIMONIOS_MIN_RATING'),
        'max_rating' => Configuration::get('GESTOR_TESTIMONIOS_MAX_RATING'),
        'submit_action' => 'submit' . $this->name
    ]);

    return $this->display(__FILE__, 'views/templates/admin/configure.tpl');
}
```

***

### 🧪 PASO 6: Instalación y Pruebas&#x20;

#### 6.1 Instalación del módulo

1. **Acceder al backoffice** de PrestaShop
2. **Ir a Módulos > Gestor de Módulos**
3. **Subir módulo** o colocarlo en `/modules/gestortestimonios/`
4. **Buscar "Gestor de Testimonios"**
5. **Hacer clic en "Instalar"**

#### 6.2 Verificación de la instalación

{% hint style="success" %}
**Comprobar:**

* ✅ Módulo aparece en la lista de módulos instalados
* ✅ Tabla `ps_testimonios` creada en la base de datos
* ✅ Configuraciones guardadas en `ps_configuration`
* ✅ Hooks registrados correctamente
{% endhint %}

#### 6.3 Pruebas básicas

1. **Configuración**: Acceder a la configuración del módulo
2. **Cambiar valores**: Modificar testimonios por página
3. **Verificar guardado**: Comprobar que se guardan los cambios
4. **Frontend**: Verificar que no hay errores en el frontend

***

### 📝 Ejercicios Prácticos

{% tabs %}
{% tab title="Ejercicio 1" %}
#### Personalización del Constructor (10 min)

Modifica el constructor para añadir:

* Campo `$this->dependencies` con módulos requeridos
* Campo `$this->limited_countries` para restricciones geográficas
{% endtab %}

{% tab title="Ejercicio 2" %}
#### Nuevo Hook

Implementa el hook `displayCustomerAccount` para mostrar un enlace en la cuenta del cliente.
{% endtab %}

{% tab title="Ejercicio 3" %}
#### Validación Avanzada&#x20;

Añade validación en `processConfiguration()` para verificar que no se pueda deshabilitar el módulo si hay testimonios pendientes.
{% endtab %}
{% endtabs %}

***

### 🔄 Resumen de la Clase

#### ✅ Lo que hemos logrado:

1. **Estructura completa** del módulo con directorios organizados
2. **Clase principal** funcional con instalación/desinstalación
3. **Base de datos** optimizada para testimonios
4. **Hooks básicos** para integración con PrestaShop
5. **Panel de administración** con configuración funcional

#### 🎯 Próxima clase:

En la **Clase 2** desarrollaremos:

* Controladores frontend completos
* Templates avanzados con formularios
* Sistema de validación y seguridad
* Funcionalidades de aprobación/rechazo
* Estilos y JavaScript
* Testing y optimización

***

### 📚 Recursos Adicionales

* [Documentación PrestaShop](https://developers.prestashop.com/)
* [Hook Reference](https://developers.prestashop.com/1.7/modules/concepts/hooks/)
* [Database API](https://developers.prestashop.com/1.7/development/database/)
* [Module Structure](https://developers.prestashop.com/1.7/modules/creation/)

***

### ❓ Preguntas Frecuentes

¿Por qué usar pSQL() en las consultas?"  `pSQL()` previene inyecciones SQL saneando los datos de entrada.

&#x20;"¿Cuándo se ejecutan los hooks?" Los hooks se ejecutan en momentos específicos del ciclo de vida de PrestaShop.

"¿Es obligatorio el archivo index.php?" Sí, es una medida de seguridad estándar de PrestaShop para prevenir acceso directo.&#x20;

***
