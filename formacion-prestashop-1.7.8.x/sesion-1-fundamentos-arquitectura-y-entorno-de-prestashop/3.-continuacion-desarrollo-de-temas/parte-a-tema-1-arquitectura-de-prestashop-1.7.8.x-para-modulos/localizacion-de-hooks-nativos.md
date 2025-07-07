# Localización de Hooks Nativos

* **Conceptos Clave:** Es fundamental saber qué hooks existen y dónde para poder extender PrestaShop.
* **Actividad en Clase / Demostración:**
  * **Método 1: Base de Datos:** Mostrar la tabla `ps_hook` en phpMyAdmin y explicar sus columnas (`name`, `title`, `description`).
  * **Método 2: Búsqueda en Código:** Realizar una búsqueda global en el proyecto PrestaShop (Core y tema) de:
    * `Hook::exec(` (para hooks ejecutados desde PHP).
    * `{hook h=...}` (para hooks en plantillas Smarty).
  * **Método 3: Modo Debug y Profiler:** En páginas Symfony, el Profiler (Barra de Debug) puede listar los eventos (muchos son hooks) que se disparan.
  * **Recurso Principal:** [Lista Oficial de Hooks en PrestaShop DevDocs](https://devdocs.prestashop-project.org/1.7/modules/concepts/hooks/list-of-hooks/) y cómo usarla.
