# Parte B: Introducción al Patrón MVC

#### ¿Qué es MVC en PrestaShop?

**MVC** es una arquitectura que separa la lógica en tres componentes interconectados:

**🗃️ Modelo (Model)**

* **Función**: Lógica de datos
* **Responsabilidad**: Interactuar con la base de datos (leer, escribir, actualizar)
* **En nuestro caso**: Clase `Configuration` de PrestaShop

**👁️ Vista (View)**

* **Función**: Presentación visual
* **Responsabilidad**: Lo que ve el usuario
* **En nuestro caso**: Archivos de plantilla (.tpl) y HelperForm

**🧠 Controlador (Controller)**

* **Función**: Lógica de negocio
* **Responsabilidad**: Recibe peticiones, comunica con el Modelo, instruye a la Vista
* **En nuestro caso**: Nuestro controlador personalizado

### Flujo MVC en el Back Office

```graphql
graph TD
A[Admin hace clic en 'Configurar'] --> B[Petición llega al Controlador]
B --> C[Controlador pide datos al Modelo]
C --> D[Modelo devuelve configuración]
D --> E[Controlador pasa datos a la Vista]
E --> F[Vista renderiza formulario HTML]
F --> G[Admin ve la página de configuración]
```

### El Método getContent(): Puerta de Entrada

Cuando un administrador hace clic en **"Configurar"**, PrestaShop ejecuta el método `getContent()` en la clase principal del módulo.

{% hint style="info" %}
**Importante**: `getContent()` es el punto de entrada estándar para la configuración de módulos en PrestaShop.
{% endhint %}
