# Actividad en Clase / Práctica Guiada:

1. **Dentro de la carpeta `modules/` de su PrestaShop (mapeada desde su proyecto local):** Cada alumno creará una nueva carpeta para el módulo que desarrollaremos a lo largo del curso: `mibloqueinfo`.
2. **Dentro de `mibloqueinfo/`, crearán:**
   * `mibloqueinfo.php` (vacío por ahora, o con la estructura mínima de la clase).
   * `config.xml` (vacío o con la estructura mínima).
   * `logo.png` (pueden copiar uno genérico). <img src="../../../../.gitbook/assets/logo.png" alt="" data-size="line">
   * Las carpetas: `views/`, `views/css/`, `views/js/`, `views/templates/`, `views/templates/hook/`, `translations/`.
   * Un archivo `index.php` de seguridad en cada una de estas carpetas.

* **Recurso:** Contenido del `index.php` de seguridad para copiar y pegar.

```php
<?php
/**
 * 2007-2023 PrestaShop
 *
 * NOTICE OF LICENSE
 *
 * This source file is subject to the Academic Free License (AFL 3.0)
 * that is bundled with this package in the file LICENSE.txt.
 * It is also available through the world-wide-web at this URL:
 * http://opensource.org/licenses/afl-3.0.php
 * If you did not receive a copy of the license and are unable to
 * obtain it through the world-wide-web, please send an email
 * to license@prestashop.com so we can send you a copy immediately.
 *
 * @author    PrestaShop SA <contact@prestashop.com>
 * @copyright 2007-2023 PrestaShop SA
 * @license   http://opensource.org/licenses/afl-3.0.php  Academic Free License (AFL 3.0)
 */

header('Expires: Mon, 26 Jul 1997 05:00:00 GMT');
header('Last-Modified: ' . gmdate('D, d M Y H:i:s') . ' GMT');

header('Cache-Control: no-store, no-cache, must-revalidate');
header('Cache-Control: post-check=0, pre-check=0', false);
header('Pragma: no-cache');

header('Location: ../../');
exit;

```

**3. Test de Conceptos**

* **Actividad:** Preguntas para afianzar los conocimientos de la sesión.
* **Ejemplo de Preguntas:**
  1. Nombra dos formas de ver los logs de PHP/Apache de tu contenedor PrestaShop.
  2. ¿Para qué sirve el archivo `config.xml` en un módulo?
  3. ¿Qué significa PSR-4 y por qué es importante para los módulos modernos?
  4. ¿Cuál es el propósito principal de la carpeta `views/templates/hook/` en un módulo?
  5. ¿Qué comando de la PrestaShop Console usarías para limpiar la caché?

***

#### 4. Feedback Individualizado y Preguntas

* **Actividad:** Sesión abierta de Q\&A.
* **Objetivo:**
  * Aclarar dudas sobre los temas de gestión del entorno (logs, caché, Git, Composer).
  * Resolver preguntas sobre la estructura de archivos y carpetas de un módulo.
  * Asegurar que todos han podido crear el esqueleto inicial de su módulo `mibloqueinfo`.
