# Proyecto Final: Módulo de Valoraciones

**Objetivo:** Permitir que los clientes dejen valoraciones (puntuación y comentario) en las fichas de producto.

Análisis de Funcionalidades y Conexión con la Sesión:

1. Formulario para enviar valoración:
   * Necesitará un `hook` en la ficha de producto (ej: `displayFooterProduct`) para mostrar el formulario.
   * El formulario enviará los datos a un `FrontController` personalizado.
2. Listado de valoraciones visibles:
   * El mismo `hook` (`displayFooterProduct`) mostrará las valoraciones ya aprobadas.
   * Para obtener las valoraciones, necesitaremos una consulta a la base de datos con `Db::getInstance()->executeS()`.
3. Panel en el Back Office para moderar:
   * ¡Aquí aplicaremos todo lo de hoy!
   * Necesitaremos una tabla personalizada en la base de datos para almacenar las valoraciones (puntuación, comentario, id\_producto, id\_cliente, estado, fecha). Esto se creará con el método `install()` (Tema #7).
   * Crearemos una clase `ObjectModel` llamada `Valoracion` para manejar cada valoración de forma sencilla (Tema #7).
   * Crearemos un controlador de Back Office con una lista de valoraciones (`HelperList`).
   * Este controlador tendrá acciones para aprobar o eliminar valoraciones, que interactuarán con nuestro `ObjectModel` (Tema #7).



### Módulo final del curso

{% file src="../../.gitbook/assets/gestortestimonios.zip" %}
