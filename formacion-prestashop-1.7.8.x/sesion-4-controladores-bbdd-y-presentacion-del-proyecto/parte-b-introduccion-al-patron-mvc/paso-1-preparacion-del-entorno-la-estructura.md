# Paso 1: Preparación del Entorno (La Estructura)

El primer paso es crear la estructura de carpetas y archivos vacíos. Esto ayuda a los alumnos a entender la organización de un módulo.

**Acción**: Crear la siguiente estructura de carpetas y archivos en la carpeta `/modules` de PrestaShop:

<pre><code>/modules
└── /quicknotes/
     ├── config.xml
     ├── quicknotes.php
<strong>     ├── /classes/
</strong><strong>     │   └── Quicknote.php
</strong>     └── /controllers/
         └── /admin/
             └── AdminQuicknotesController.php
</code></pre>
