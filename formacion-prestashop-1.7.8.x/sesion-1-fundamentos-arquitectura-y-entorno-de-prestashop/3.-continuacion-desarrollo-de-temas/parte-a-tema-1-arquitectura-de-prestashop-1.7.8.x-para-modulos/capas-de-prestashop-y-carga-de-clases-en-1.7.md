# Capas de PrestaShop y Carga de Clases en 1.7

* **Conceptos Clave:**
  * PrestaShop se organiza en capas principales: **Core** (con su lógica de negocio y entidades), **Módulos** (para extender funcionalidades) y **Temas** (para la presentación).
  * Dentro del Core, conviven directorios tradicionales como `/classes` (ObjectModels, Helpers legacy) y `/controllers` (AdminControllers y FrontControllers legacy) con el directorio `/src` que alberga la nueva arquitectura basada en Symfony (servicios, adaptadores, nuevos controladores).
  * La **carga de clases (Autoloading)** en PrestaShop 1.7 es mixta:
    * Un autoloader legacy para las clases antiguas.
    * **Composer y el estándar PSR-4** para las clases con namespaces (principalmente en `/src` del Core y en los módulos modernos que también usen un directorio `src/`).
* **Actividad en Clase / Ejemplo Práctico:**
  * Navegaremos brevemente por la estructura de directorios de una instalación de PrestaShop.
  * Mostraremos cómo se instancia una clase Core legacy (ej. `Product`) y cómo un módulo moderno puede usar namespaces para sus propias clases (ej. `MiModulo\Service\MiServicio`).

<figure><img src="../../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>
