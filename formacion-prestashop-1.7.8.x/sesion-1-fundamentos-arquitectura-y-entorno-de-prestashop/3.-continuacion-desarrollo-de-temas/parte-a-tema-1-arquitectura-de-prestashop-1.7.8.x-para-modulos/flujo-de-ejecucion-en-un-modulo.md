# Flujo de Ejecución en un Módulo

* **Conceptos Clave:**
  * Un módulo "cobra vida" principalmente a través de:
    1. **Instalación/Desinstalación:** Métodos `install()` y `uninstall()`.
    2. **Hooks:** Se registran en `install()` y PrestaShop llama a los métodos `hookNombreDelHook()` correspondientes cuando se ejecuta ese hook en el Core o una plantilla.
    3. **Controladores del Módulo:** Cuando se accede a una URL específica del módulo, se instancia su `ModuleFrontController` o `ModuleAdminController`.



<figure><img src="../../../../.gitbook/assets/Flujo de ejecución.png" alt=""><figcaption></figcaption></figure>
