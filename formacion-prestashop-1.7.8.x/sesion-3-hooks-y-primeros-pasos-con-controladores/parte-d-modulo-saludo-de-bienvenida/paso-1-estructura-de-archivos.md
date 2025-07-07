# Paso 1: Estructura de Archivos

### Crear la Estructura Base

Navega a `/modules/` y crea la siguiente estructura:

```
welcomelogin/
├── config.xml
├── welcomelogin.php
├── views/
│   ├── css/
│   │   └── welcomelogin.css
│   └── templates/
│       └── hook/
│           └── home.tpl
└── index.php (en cada directorio)
```

#### Archivos de Seguridad

{% hint style="warning" %}
**Importante**: Recuerda crear el archivo `index.php` de seguridad en cada directorio usando el contenido estándar de PrestaShop que vimos en el módulo anterior.
{% endhint %}
