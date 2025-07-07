# Paso 1: Estructura de Archivos

### Creación de Directorios

Navega a `/modules/` en tu instalación de PrestaShop y crea la siguiente estructura:

```
productpersonalization/
├── views/
│   └── templates/
│       └── admin/
└── index.php
```

### 🔒 Archivos de Seguridad

{% hint style="warning" %}
**¡Muy Importante!** Cada directorio debe contener un archivo `index.php` de seguridad para prevenir acceso directo.
{% endhint %}

Crea un archivo `index.php` en **cada directorio** con el contenido visto en ejercicios anteriores
