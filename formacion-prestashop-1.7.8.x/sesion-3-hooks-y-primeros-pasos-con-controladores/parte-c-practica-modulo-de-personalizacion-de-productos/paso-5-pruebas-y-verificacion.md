# Paso 5: Pruebas y Verificación

#### Lista de Verificación

* [ ] ✅ **Estructura de archivos** creada correctamente
* [ ] ✅ **Archivos de seguridad** en todos los directorios
* [ ] ✅ **Módulo instalable** sin errores
* [ ] ✅ **Campos aparecen** en páginas de producto
* [ ] ✅ **Panel de configuración** funcional
* [ ] ✅ **Desinstalación limpia** sin residuos

#### Pasos de Prueba

1. **Instalación**: Ve a `Módulos > Módulos y Servicios` y busca tu módulo
2. **Configuración**: Accede al panel y personaliza las etiquetas
3. **Frontend**: Verifica que los campos aparecen en productos
4. **Funcionalidad**: Añade un producto personalizado al carrito
5. **Desinstalación**: Confirma que se eliminan todos los campos

### Notas Técnicas

{% hint style="info" %}
**Limitación Conocida**: PrestaShop no soporta `maxlength` nativamente en campos de personalización. Para implementarlo, necesitarías JavaScript adicional.
{% endhint %}

{% hint style="success" %}
**Buena Práctica**: Usar constantes para claves de configuración previene errores de escritura y facilita el mantenimiento.
{% endhint %}
