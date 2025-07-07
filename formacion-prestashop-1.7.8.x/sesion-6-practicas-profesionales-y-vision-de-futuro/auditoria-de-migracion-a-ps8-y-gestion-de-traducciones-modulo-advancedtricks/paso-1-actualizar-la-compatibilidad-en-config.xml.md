# Paso 1: Actualizar la Compatibilidad en config.xml

Es el primer paso formal para indicar que nuestro módulo está preparado para versiones más recientes.

*   Acción: Abre el archivo `modules/advancedtricks/config.xml` y modifica la etiqueta `<ps_versions_compliancy>`.

    XML

    ```xml
    <ps_versions_compliancy>
        <min>1.7.0.0</min>
        <max>8.99.99</max> </ps_versions_compliancy>
    ```
