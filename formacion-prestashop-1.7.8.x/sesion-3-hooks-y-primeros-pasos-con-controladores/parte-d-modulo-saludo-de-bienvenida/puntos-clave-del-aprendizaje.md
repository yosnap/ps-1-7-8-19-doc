# 💡 Puntos Clave del Aprendizaje

### Action Hooks vs Display Hooks

{% hint style="info" %}
**Action Hooks** (`actionAuthentication`): Se usan para ejecutar lógica de fondo, como guardar datos. No devuelven contenido HTML.

**Display Hooks** (`displayHome`): Se usan para mostrar contenido visual. Devuelven HTML que se insertará en la página.
{% endhint %}

### Gestión de Base de Datos

{% hint style="success" %}
**Buena Práctica**: Siempre usa `_DB_PREFIX_` para los nombres de tabla y `_MYSQL_ENGINE_` para el tipo de motor. Esto asegura compatibilidad con diferentes configuraciones.
{% endhint %}

### Seguridad en Consultas

{% hint style="warning" %}
**Importante**: En un entorno de producción, siempre usa consultas preparadas o valida los datos antes de usarlos en SQL para prevenir inyecciones.
{% endhint %}
