# Paso 2: Archivo de Configuración

Crea el archivo `config.xml` en la raíz del módulo:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<module>
    <name>welcomelogin</name>
    <displayName><![CDATA[Módulo Saludo de Bienvenida]]></displayName>
    <version><![CDATA[1.0.0]]></version>
    <description><![CDATA[Muestra un saludo personalizado a los clientes que inician sesión, indicando su última visita.]]></description>
    <author><![CDATA[Desarrollo PS]]></author>
    <tab><![CDATA[front_office_features]]></tab>
    <is_configurable>0</is_configurable>
    <need_instance>1</need_instance>
    <ps_versions_compliancy>
        <min>1.7.0.0</min>
        <max>1.7.8.99</max>
    </ps_versions_compliancy>
</module>
```
