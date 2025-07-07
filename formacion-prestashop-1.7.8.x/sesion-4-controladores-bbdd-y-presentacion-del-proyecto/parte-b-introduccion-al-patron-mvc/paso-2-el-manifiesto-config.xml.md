# Paso 2: El Manifiesto (config.xml)

Este archivo es la "tarjeta de presentación" del módulo para PrestaShop.

**Acción**: Crear el archivo `config.xml` y pegar el siguiente código:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<module>
    <name>quicknotes</name>
    <displayName><![CDATA[Notas Rápidas]]></displayName>
    <version><![CDATA[1.0.0]]></version>
    <description><![CDATA[Añade una sección de notas rápidas en el Back Office.]]></description>
    <author><![CDATA[Desarrollo PS]]></author>
    <tab><![CDATA[administration]]></tab>
    <is_configurable>1</is_configurable> </module>
```

