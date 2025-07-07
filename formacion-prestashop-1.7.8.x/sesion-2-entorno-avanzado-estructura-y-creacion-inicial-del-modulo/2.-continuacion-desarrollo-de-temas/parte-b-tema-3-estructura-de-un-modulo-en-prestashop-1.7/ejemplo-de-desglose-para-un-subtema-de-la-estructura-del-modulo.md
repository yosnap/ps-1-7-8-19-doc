# Ejemplo de desglose para un subtema de la Estructura del Módulo:

**Archivos Clave: `.php` principal, `config.xml`, `logo.png`, `translations/`, `views/`**

* **Conceptos Clave:**
  * **`nombremodulo.php`:** El corazón del módulo, contiene la clase principal que hereda de `Module`. Define propiedades, métodos de instalación/desinstalación y hooks.
  * **`config.xml`:** Define metadatos del módulo para PrestaShop (nombre, versión, autor, etc.). Se puede autogenerar o crear manualmente.
  * **`logo.png` (o `.gif`):** Icono visible en la lista de módulos.
  * **`translations/`:** Carpeta para los archivos de traducción (ej. `es.php`).
  * **`views/`:** Contiene plantillas (`.tpl`), CSS y JS para el módulo.
* **Actividad en Clase / Ejemplo Práctico:**
  * Crear un archivo `mibloqueinfo.php` vacío y empezar a rellenar el constructor con las propiedades básicas (`$this->name`, `$this->version`, etc.).
  * Crear un `config.xml` básico manualmente.
  * Mostrar dónde colocar un `logo.png`.
  * Crear la estructura de carpetas `views/templates/hook/` y `translations/`.
* **Recurso Visual / "Infografía" Sugerida:**

```
mi_modulo/
├── config.xml                      # Metadatos del módulo para PrestaShop.
├── mi_modulo.php                   # Archivo principal con la lógica del módulo.
├── logo.png                        # Icono que se muestra en la lista de módulos.
├── classes/                        # Para tus clases PHP personalizadas (Modelos, etc.).
│   └── MiClaseModelo.php
├── controllers/                    # Controladores para páginas específicas.
│   ├── admin/                      # Controladores para el Back Office.
│   │   └── AdminMiModuloController.php
│   └── front/                      # Controladores para el Front Office.
├── translations/                   # Archivos de traducción (es.php, en.php, etc.).
│   └── es.php
├── upgrade/                        # Scripts para actualizar el módulo entre versiones.
│   └── upgrade-1.1.0.php
└── views/                          # Contiene las plantillas y otros recursos visuales.
    ├── css/                        # Hojas de estilo CSS.
    │   └── front.css
    ├── js/                         # Archivos JavaScript.
    │   └── front.js
    ├── imgs/                       # Imágenes utilizadas por el módulo.
    │   └── mi_imagen.png
    └── templates/                  # Plantillas Smarty (.tpl).
        ├── admin/                  # Plantillas para el Back Office.
        │   └── configure.tpl
        └── hooks/                  # Plantillas para los hooks.
            └── miHook.tpl
```

*   **Código de Ejemplo (`mibloqueinfo.php` inicial):**

    ```php
    <?php
    if (!defined('_PS_VERSION_')) { exit; }
    class MiBloqueInfo extends Module {
        public function __construct() {
            $this->name = 'mibloqueinfo';
            $this->tab = 'front_office_features';
            // ... más propiedades
            parent::__construct();
            $this->displayName = $this->l('Mi Bloque de Info');
            // ...
        }
    }
    ```

_(...y así para cada subtema de la "Estructura de un Módulo")_
