# Paso 2: Auditoría del Registro de Hooks

*   El Código a Revisar (`install()`):

    PHP

    ```php
    public function install() {
        return parent::install() && $this->registerHook('displayHome');
    }
    ```
* Análisis: Este método funciona perfectamente en PrestaShop 8 por retrocompatibilidad. Sin embargo, se considera "legacy".
* Explicación: La forma moderna en PS8 es declarar los hooks en un archivo de configuración (`config/services.yml`) dentro del módulo. Allí se definen como "servicios" que se "etiquetan" con el nombre del hook. Esto sigue los patrones de Symfony y permite a PrestaShop gestionar los módulos de forma más eficiente sin tener que ejecutar el método `install()` para saber a qué hooks está suscrito.

**Detalle:**

El método "moderno" en PS8 (no es necesario que lo cambies, es solo para que lo veas): En lugar de `registerHook()`, crearías un archivo en `modules/advancedtricks/config/services.yml` con este contenido:

YAML

```
services:
  # Definimos nuestro módulo como un servicio
  advancedtricks.module:
    class: AdvancedTricks
    # ... otras configuraciones del servicio ...

  # Definimos el hook como otro servicio
  advancedtricks.hook.display_home:
    class: PrestaShop\PrestaShop\Core\Module\HookListener
    arguments:
      - "@advancedtricks.module" # Le decimos a qué módulo pertenece
      - "hookDisplayHome"      # El método que debe llamar
    tags:
      - { name: 'kernel.event_listener', event: 'hook.displayHome' } # Se suscribe al hook
```

Como puedes ver, el método moderno es más verboso y está orientado a servicios, siguiendo la arquitectura de Symfony.
