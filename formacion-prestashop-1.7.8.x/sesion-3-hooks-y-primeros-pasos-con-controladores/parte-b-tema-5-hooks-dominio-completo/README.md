# Parte B: Tema #5 - Hooks: Dominio Completo

**¿Qué son los Hooks?**

Los Hooks son el sistema de "enchufes" de PrestaShop. El núcleo de la aplicación, en puntos clave de su ejecución (como al cargar la cabecera, al mostrar un producto, o al actualizar un carrito), ofrece un "hook". Tu módulo puede "registrarse" en ese hook para ejecutar un trozo de tu propio código en ese preciso instante. Es la forma más limpia y modular de extender PrestaShop sin modificar su núcleo.

**Implementando nuestros primeros Hooks**

Una vez que hemos registrado un hook en el método `install()`, debemos crear la función que se ejecutará. El nombre de la función siempre sigue el formato: `public function hookNombreDelHook($params)`.
